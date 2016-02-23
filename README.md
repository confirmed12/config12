local meta = FindMetaTable("Player")

if!(GAMEMODE.DarkRPFinishedLoading) then
	if(SERVER) then
		print("[DarkRP Advanced Inv V2]: The addon will not load because you're not running a valid DarkRP(Outdated Or not even running)")
	end
	return
end

if(SERVER) then
	util.AddNetworkString("drp_advi2_syncitems")
	util.AddNetworkString("drp_advi2_item_changed")
	util.AddNetworkString("drp_advi2_open_inv_fcl")
	util.AddNetworkString("drp_advi2_open_inv_tcl")
	util.AddNetworkString("drp_advi2_open_cd_holster")
	resource.AddSingleFile("materials/advi2_icons/inv2_destroy.png")
	resource.AddSingleFile("materials/advi2_icons/inv2_drop.png")
	resource.AddSingleFile("materials/advi2_icons/inv2_use.png")
	MySQLite.query("CREATE TABLE IF NOT EXISTS darkrp_advinv2 ( unique_id varchar(255), inv longtext, log longext, PRIMARY KEY(unique_id) )")
end

SH_ADVANCED_INV_V2 = {}
SH_ADVANCED_INV_V2["Settings"] = {
	default_inv_size = 9 * 3, // 9x3 Default inventory slots(unless the player is in a user-group, look below)
	enable_holster = true, // Enable/Disable the /holster command, false = disable, true = enable
	enable_new_drop = true, // Enable/Disable the custom /drop command, true = enable, false = disabled
	shipment_model = "models/Items/item_item_crate.mdl", // Shipment Model
	starting_items = {
		//The Items new players spawn with, Add the item classes here, example: "food_glassbottle",
		"medkit",
		"medkit",
		"food_glassbottle",
		"food_glassbottle",
		"food_glassbottle",
	},
	inventory_hud = { //Customize the Box saying "Press SHIFT-B To open Your Inventory"
		enabled = true, // True = Enabled, False = Disabled
		position = "bottom", // Valid Positions: Bottom, Top
	},
	cops_can_unholster_weapons = false, //True=Cops can unholster weapons, false = they can't
}
if(SERVER) then
	SH_ADVANCED_INV_V2["Settings"].default_inv_size_special = function(ply)
	if(ply:IsUserGroup("owner")) then
		return 9 * 10
	end
	if(ply:IsUserGroup("superadmin")) then
		return 9 * 7
	end 
	if(ply:IsUserGroup("admin")) then
		return 9 * 6
	end
	if(ply:IsUserGroup("donator") or ply:IsUserGroup("premium") or ply:IsUserGroup("VIP") or ply:IsUserGroup("vip")) then
		return 9 * 5
	end
	return SH_ADVANCED_INV_V2["Settings"].default_inv_size
	end
end
SH_ADVANCED_INV_V2["Items"] = {}
SH_ADVANCED_INV_V2["Item_Categories"] = {}
SH_ADVANCED_INV_V2["Valid_Containers"] = {}


function isValidInvItem(class)
	if(class) then
		for k,v in pairs(SH_ADVANCED_INV_V2.Items) do
			if(v.class == class) then
				return true
			end
		end 
	end
	return false
end

function isItemContainer(class)
	if(class) then
		if(isValidInvItem(class)) then
			if(#SH_ADVANCED_INV_V2["Valid_Containers"] > 0) then
				for k,v in pairs(SH_ADVANCED_INV_V2["Valid_Containers"]) do
					if(v.class == class) then
						return true
					end
				end
			end
		end
	end
	return false
end

function GetItemContainerSize(class)
	if(class) then
		if(isItemContainer(class)) then
			for k,v in pairs(SH_ADVANCED_INV_V2["Valid_Containers"]) do
				if(v.class == class) then
					return v
				end
			end
		end
	end
	return {class = false, size = false}
end


function SH_ADVANCED_INV_V2.PreAddItemCategory(name, col, icon)
	if(name) then
		if!(col) then
			col = Color(255,255,255)
		end
		if!(icon) then
			icon = "icon16/box.png"
		end
		table.insert(SH_ADVANCED_INV_V2["Item_Categories"], {name = name, col = col, icon = icon})
		if(SERVER) then
			print("[DarkRP Advanced Inv V2] Added Category: " .. name)
		end
	end
end

//Categories, Add them before adding items to it
SH_ADVANCED_INV_V2.PreAddItemCategory("Weapons", Color(255, 0, 0), "icon16/gun.png")
SH_ADVANCED_INV_V2.PreAddItemCategory("Foods", Color(0, 255, 0), "icon16/basket.png")
SH_ADVANCED_INV_V2.PreAddItemCategory("Ammunition", Color(255, 255, 0), "icon16/brick_add.png")
SH_ADVANCED_INV_V2.PreAddItemCategory("Containers", Color(0, 255, 0), "icon16/package.png")
SH_ADVANCED_INV_V2.PreAddItemCategory("Medical Supplies", Color(0, 255, 255), "icon16/pill.png")
SH_ADVANCED_INV_V2.PreAddItemCategory("Miscellaneous", Color(255, 255, 255), "icon16/brick.png")



function SH_ADVANCED_INV_V2.AddItem(name, desc,  color1, color2, func_use, func_destroy, func_drop, can_use, can_drop, category, class_name, model, max, buy, custom, teams)
	if!(name and desc and color1 and color2 and func_use and func_destroy and func_drop and can_use and can_drop and category and class_name and model and max and buy and custom and teams) then
		if(CLIENT) then return end
		print("[DarkRP Advanced Inv V2] Trying to add a corrupted item ("..name..")!")
		return
	end
	if(SERVER) then
		print("[DarkRP Advanced Inv V2] Registered Item: " .. name)
	end
	if(isValidInvItem(class_name)) then
		return
	end
	SH_ADVANCED_INV_V2["Items"][#SH_ADVANCED_INV_V2["Items"] + 1] = {
		name = tostring(name),
		desc = tostring(desc),
		color1 = color1,
		color2 = color2,
		category = category,
		onuse = func_use,
		ondrop = func_drop,
		ondestroy = func_destroy,
		canuse = can_use,
		candrop = can_drop,
		teams = teams,
		class = class_name,
		model = model,
		max = max,
		buy = buy,
		customcheck = custom
	}
end

function Advi2ClassToID(class)
	for k,v in pairs(SH_ADVANCED_INV_V2["Items"]) do
		if(v.class == class) then
			return k
		end
	end
	return false
end

if(CLIENT) then

end
DarkRP.removeChatCommand("buy")
DarkRP.removeChatCommand("buyshipment")




function Getinv2ItemInfo(class)
	if(class) then
		if(isValidInvItem(class)) then
			for k,v in pairs(SH_ADVANCED_INV_V2.Items) do
				if(string.lower(tostring(v.class)) == string.lower(tostring(class))) then
					return v
				end
			end
		end
	end
end


//Load Files
local files_inc_items = file.Find("darkrp_modules/darkrpadvi2/sh_items/*", "LUA")
for k, v in pairs(files_inc_items) do
if(SERVER) then
	AddCSLuaFile("darkrp_modules/darkrpadvi2/sh_items/" .. v)	
end
include("darkrp_modules/darkrpadvi2/sh_items/" .. v)	
end

local files_inc_plugins = file.Find("darkrp_modules/darkrpadvi2/plugins/*", "LUA")
for k, v in pairs(files_inc_plugins) do
AddCSLuaFile("darkrp_modules/darkrpadvi2/plugins/" .. v)	
include("darkrp_modules/darkrpadvi2/plugins/" .. v)
print("Loaded Plugin: "..v)
end


//Chat Commands
if(SH_ADVANCED_INV_V2["Settings"].enable_holster) then
DarkRP.declareChatCommand{
	command = "holster",
	description = "Holster your weapon",
	delay = 1.5
}
end



if(SERVER) then


local function CanHolster(ply, wep)
	if(ply and ply:IsValid() and ply:Alive() and wep) then
		if(ply:Team() and RPExtraTeams and RPExtraTeams[ply:Team()] and RPExtraTeams[ply:Team()].weapons) then
			local found = false
			for k,v in pairs(RPExtraTeams[ply:Team()].weapons) do
				if(string.lower(tostring(v)) == string.lower(tostring(wep))) then
					found = true
				end
			end
			if(found) then
				return false
			else
				return true
			end
		end
	end
	return true
end

local CanDrop
CanDrop = CanHolster


if(SH_ADVANCED_INV_V2["Settings"].enable_new_drop) then


local function PlayerDropWeapon(ply, wep, class)
	if!(Getinv2ItemInfo(class)) then
		return
	end
	local info = Getinv2ItemInfo(class)

    local ent = ents.Create("advi2_item")
    ent:SetPos(ply:GetShootPos() + ply:GetAimVector() * 30)
    ent:SetModel(info.model)
    ent.dt.item = Advi2ClassToID(class)
	ent.iclass = class
	ent.dt.item = Advi2ClassToID(class)
    ent:Spawn()
    ent:Activate()
    wep:Remove()
end 






timer.Simple(5, function()
DarkRP.removeChatCommand("drop")
DarkRP.removeChatCommand("makeshipment")
DarkRP.declareChatCommand{
	command = "drop",
	description = "Drops your current weapon",
	delay = 1.5
}

DarkRP.defineChatCommand("drop", function(ply)
	if(ply and ply:IsValid() and ply:IsPlayer() and ply:Alive() and ply.advi and ply:GetActiveWeapon()) then
		local wep = ply:GetActiveWeapon()
		if(isValidInvItem(wep:GetClass())) then
			local class = tostring(wep:GetClass())
			if(GAMEMODE.Config and GAMEMODE.Config.DisallowDrop) then
				for k,v in pairs(GAMEMODE.Config.DisallowDrop) do
					if(string.lower(tostring(k)) == string.lower(class)) then
						if(v) then
							ply:ChatPrint("You can't drop this weapon.")
							return ""
						end
					end
				end
			end
			if(CanDrop(ply, class)) then
				if!(Getinv2ItemInfo(class).candrop(ply, class)) then
					ply:ChatPrint("You can't drop this weapon.")
					return ""
				end
				local canDrop = hook.Call("canDropWeapon", GAMEMODE, ply, wep)
				if!(canDrop) then
					return ""
				end
				local RP = RecipientFilter()
    			RP:AddAllPlayers()

  				umsg.Start("anim_dropitem", RP)
       				umsg.Entity(ply)
    			umsg.End()
    			ply.anim_DroppingItem = true
    			timer.Simple(1, function()
       				if IsValid(ply) and IsValid(wep) and wep:GetModel() and wep:GetModel() ~= "" then
            			PlayerDropWeapon(ply, wep, class)
        			end
    			end)
			end
		end
	end
end)
end)
end





if(SH_ADVANCED_INV_V2["Settings"].enable_holster) then
DarkRP.defineChatCommand("holster", function(ply)
	if(ply and ply:IsValid() and ply:IsPlayer() and ply:Alive() and ply.advi and ply:GetActiveWeapon()) then
		local wep = ply:GetActiveWeapon()
		if(isValidInvItem(wep:GetClass())) then
			if(ply:AdviCanHaveMoreOf(wep:GetClass(), 1)) then
				if(ply:HasInv2Space(1)) then
					if!(CanHolster(ply, tostring(wep:GetClass()))) then
						ply:ChatPrint("You can not holster this weapon!")
						return ""
					end
					ply.sn_holster_cd = true
					net.Start("drp_advi2_open_cd_holster")
					net.Send(ply)
					timer.Simple(2, function()
						ply:AddinvItem(wep:GetClass(), 1)
						wep:Remove()
						ply:ChatPrint("You've holstered your weapon.")
						ply:SelectWeapon("keys")
						ply.sn_holster_cd = false
					end)
					return
					else
					ply:ChatPrint("You don't have enough space in your inventory!")
						return
					end
				else
					ply:ChatPrint("You can only have "..info["max"].."x of this item in your inventory.")
				return
			end
		end
	end
end)

hook.Add("PlayerSwitchWeapon", "sn_advi_holstert", function(ply)
	if(ply.sn_holster_cd) then
		return true
	end
end)



end

end



CustomShipments = {}
if(SERVER) then

function meta:GetInvSize()
	local size
	local count = 0
	if(self and self.advi) then
		if(SH_ADVANCED_INV_V2["Settings"]) then
			size = SH_ADVANCED_INV_V2["Settings"].default_inv_size_special(self)
		end
	end
	if!(size) then
		size = SH_ADVANCED_INV_V2["Settings"]["default_inv_size"]
	end
	for k,v in pairs(self.advi) do
		if(v) then
			if(isItemContainer(v)) then
				size = size + GetItemContainerSize(v)["size"] or 0
			end
		end
	end
	for k,v in pairs(self.advi) do
		if(v) then
			count = count + 1
		end
	end

	if(tonumber(count) > tonumber(size)) then
		return tonumber(count)
	else
		return tonumber(math.floor(size))
	end
end



function meta:SaveAdvinv()
	if(self and self.advi) then
		local advi = util.TableToJSON(self.advi)
		local sid = sql.SQLStr(self:SteamID())
		MySQLite.query("UPDATE darkrp_advinv2 SET inv='"..advi.."' WHERE unique_id="..sid, function(res) end)
		return	
	end
end

function meta:GetAvailableInvSlot(where)
	if(where == 1) then
		local found = false
		local start = #self.advi
		local done = 0
		while(found == false) do
			if!(self.advi[start-done]) then
				return start-done
			else
				done = done + 1
			end
			if(done == start) then
				return false
			end
		end
	else
		local found = false
		local start = 1
		local done = 0
		while(found == false) do
			if!(self.advi[start+done]) then
				return start+done
			else
				done = done + 1
			end
			if(done == #self.advi) then
				return false
			end
		end
	end 
end

function meta:HasInv2Item(class)
	if(class) then
		if(isValidInvItem(class)) then
		local x = {}
			for k,v in pairs(self.advi) do
				if(v == class) then
					table.insert(x, {item = v, key = k})
				end
			end
			return x
		end
	end
	return false
end

function meta:AdviCanHaveMoreOf(class, many)
	if(class and many) then
		if(isValidInvItem(class)) then
			if(self:HasInv2Item(class)) then
				local have = self:HasInv2Item(class)

				local max = Getinv2ItemInfo(class)["max"]
				if(max < 1) then
					max = 999999
				end
				if(tonumber(#have) + tonumber(many) <= tonumber(max)) then
					return true
				else
					return false
				end
			end
			return true
		end
	end
end

function meta:AddinvItem(class, amount, skip_save)
	if!(skip_save) then skip_save = false end
	if(class) then
		if!(amount) then
			amount = 1 // Why am i even adding this idiotic bailout
		end
		if(isValidInvItem(class)) then
		//	if(#self.advi + tonumber(amount) <= self:GetInvSize()) then
			for i = 1, amount do
				local next = self:GetAvailableInvSlot(2)
				if!(next) then
					break;
				end
				self.advi[next] = tostring(class)	
			end	
		//	end
		if!(skip_save) then
			self:SaveAdvinv()
		end
		if(#self.advi < self:GetInvSize()) then
			while(#self.advi < self:GetInvSize()) do
				self.advi[#self.advi + 1] = false
			end
		else
			//Nothing
		end
		for k,v in pairs(self.advi) do
			if!(isValidInvItem(v)) then
				self.advi[k] = false
			end
		end
		return
	else
		print("[DarkRP Advanced Inv V2] Trying to give a player a invalid item(item class: "..class..", player: "..self:Nick()..")")
		end
	end
end

function meta:AdvInv2Item_Use(class, amount, pref)
	if(class) then
		if!(amount and tonumber(amount)) then
			amount = 1
		end
		if!(pref) then
			pref = false
		end
		if!(isValidInvItem(class)) then
			for k,v in pairs(self.advi) do
				if(v == class) then
					v = false
				end
			end
			return false // Failed
		end
		if(self:HasInv2Item(class)) then
			local items = self:HasInv2Item(class)
			if(tonumber(amount) > tonumber(#items)) then
				return self:ChatPrint("You don't have that many")
			else
				local info = Getinv2ItemInfo(class)

				if(info["canuse"](self, class)) then
					if(tobool(SH_ADVANCED_INV_V2["Settings"]["cops_can_unholster_weapons"]) == false) then
						if(self:isCP() and tostring(string.lower(info["category"])) == "weapons") then
							self:ChatPrint("Your job can't unholster weapons")
							return false
						end
					end

					for i = 1, tonumber(amount) do
						info["onuse"](self, class)

					end
					if(pref) then
						table.remove(self.advi, pref)
					end
					self:SaveAdvinv()
					return
				end
			end
		end
	end
	return
end

function meta:RemoveInv2Item(class, amount)
	if(class and amount) then
		local c = 0
		for k,v in pairs(self.advi) do
			if(v == class) then
				v = false
				c = c + 1
				if(c == amount) then
					break;
				end
			end
		end
	end
end

function meta:HasInv2Space(x)
	if(self and self:IsValid() and self:IsPlayer() and self.advi) then
		local slots = 0
		for k,v in pairs(self.advi) do
			if(v == false) then
				slots = slots + 1
				if(slots == x) then
					return true
				end
			end
		end
	end
	return false
end

concommand.Add("_drp_advi_useitem", function(ply, c, a)
	if(ply and ply:Alive() and ply:IsValid() and ply:IsPlayer()) then
		if(a and a[1] and a[2] and a[3] and #a == 3) then
			if(tostring(a[1]) and tonumber(a[2])) then
				local item = tostring(a[1])
				local amount = tonumber(a[2])
				local pref = tonumber(a[3])
				if(amount > 0 and amount < 999) then
					if(isValidInvItem(item)) then
						if(ply:HasInv2Item(item, 1)) then
							if(ply.advi[pref] == item) then
								if(ply:isArrested()) then return end
								ply:AdvInv2Item_Use(item, amount, pref)
								return ""
							end
							ply:AdvInv2Item_Use(item, amount, false)
							return
						end
					end
				end
			end
		end
	end
end)

concommand.Add("_drp_advi_destritem", function(ply, c, a)
	if(ply and ply:Alive() and ply:IsValid() and ply.advi) then
		if(a and a[1] and a[2]) then
			if(tostring(a[1]) and tonumber(a[2])) then
				local item = tostring(a[1])
				local ind = tonumber(a[2])
				if(ply.advi[ind] == item) then
					ply.advi[ind] = false
					ply:SaveAdvinv()
					ply:ChatPrint("You've Destroyed 1x "..Getinv2ItemInfo(item)["name"])
					return
				end
			end
		end
	end
end)

concommand.Add("_drp_advi_dropitem", function(ply, c, a)
	if(ply and ply:Alive() and ply:IsValid() and ply.advi) then
		if(a and a[1] and a[2]) then
			if(tostring(a[1]) and tonumber(a[2])) then
				local item = tostring(a[1])
				local ind = tonumber(a[2])
				if(ply.advi[ind] == item) then
					//ply.advi[ind] = false
					print(ply:isCP())
					if(tobool(SH_ADVANCED_INV_V2["Settings"]["cops_can_unholster_weapons"]) == false) then
						if(ply:isCP() and tostring(string.lower(Getinv2ItemInfo(item)["category"])) == "weapons") then
							ply:ChatPrint("Your job can't drop weapons")
							return false
						end
					end
					Getinv2ItemInfo(item).ondrop(ply, item)
					ply:ChatPrint("You've Dropped 1x "..Getinv2ItemInfo(item)["name"])
					ply.advi[ind] = false
					ply:SaveAdvinv()
					return
				end
			end
		end
	end
end)




concommand.Add("drp_advi_clearinvs", function(ply, c, a)
	if(ply:IsSuperAdmin()) then
		MySQLite.query("DELETE FROM darkrp_advinv2", function() end)
		for k,v in pairs(player.GetAll()) do
			v:LoadAdvinv2()
		end
		return ply:ChatPrint("You've removed all the inventories")
	end
end)

function meta:LoadAdvinv2()
	if(self) then
		self.advi = {}
		local sid = sql.SQLStr(self:SteamID())
		MySQLite.query("SELECT * FROM darkrp_advinv2 WHERE unique_id=" .. sid, function(res)
			if!(res) then
				local newi = {}
				for i = 1, self:GetInvSize() do
					newi[i] = false
				end
				for k,v in pairs(SH_ADVANCED_INV_V2["Settings"].starting_items) do
					if(isValidInvItem(v)) then
						newi[k] = v
					end
				end
				MySQLite.query("INSERT INTO darkrp_advinv2(unique_id, inv, log) VALUES("..sid..", '"..util.TableToJSON(newi).."', '"..util.TableToJSON({}).."')", function(res) 
					return self:LoadAdvinv2()
				end)
			else
				res = res[1]
				local inv, log = util.JSONToTable(res.inv), util.JSONToTable(res.log)
				self.advi = inv
				if(#self.advi < self:GetInvSize()) then
					while(#self.advi < self:GetInvSize()) do
						self.advi[#self.advi + 1] = false
					end
				end
				for k,v in pairs(self.advi) do
					if!(isValidInvItem(v)) then
						self.advi[k] = false
					end
				end
				self.advi_log = log
				self:PrintMessage(1, "[DarkRP Advanced Inv V2] Inventory Initialized")
				return
			end
		end)
	end
end





hook.Add("PlayerInitialSpawn", "SH_ADVANCED_INV_V2_INSP", function(ply)
	ply:LoadAdvinv2()
end)


net.Receive("drp_advi2_open_inv_fcl", function(b, ply, ...)
	if(ply and ply:IsValid() and ply:IsPlayer() and ply.advi and ply:Alive()) then
		net.Start("drp_advi2_open_inv_tcl")
		net.WriteTable(ply.advi)
		net.WriteInt( tonumber(ply:GetInvSize()) or 0, 32 )
		net.Send(ply)
	end
end)

net.Receive("drp_advi2_item_changed", function(b, ply, ...)
	if(ply and ply:IsValid() and ply:IsPlayer() and ply.advi and ply:Alive()) then
		local rec = net.ReadTable()
		if(rec["from"] and rec["to"]) then
			local from = rec["from"]
			local to = rec["to"]
			if(ply.advi[to] == false) then
				ply.advi[to] = ply.advi[from]
				ply.advi[from] = false
				ply:SaveAdvinv()
			end
		end
	end
end)




end

local function formatNumber(n)
	if (!n) then
		return 0
	end
	if n >= 1e14 then return tostring(n) end
    n = tostring(n)
    sep = sep or ","
    local dp = string.find(n, "%.") or #n+1
	for i=dp-4, 1, -3 do
		n = n:sub(1, i) .. sep .. n:sub(i+1)
    end
    return n
end


if(CLIENT) then

surface.CreateFont( "sn_advi2font_uh", {
	font = "Roboto Cn",
	size = 26,
	weight = 300,
	blursize = 0,
	scanlines = 0,
	antialias = true,
	underline = false,
	italic = false,
	strikeout = false,
	symbol = false,
	rotary = false,
	shadow = false,
	additive = false,
	outline = false,
})


local uh_status = false
local uh_count = 1
local uh_vs = 1
hook.Add("HUDPaint", "sn_draw_cd_holster", function()
if(uh_status) then
local x, y = ScrW() / 2, ScrH() / 1.5
local dots = ""
for i = 1, uh_count do
	dots = dots .. "."
end
draw.SimpleTextOutlined("Holstering."..dots, "sn_advi2font_uh", x,y, Color(255,255,255, 255), TEXT_ALIGN_CENTER, TEXT_ALIGN_CENTER, 1, Color(1,1,1))
end
end)




net.Receive("drp_advi2_open_cd_holster", function(b, ply, c)
uh_status = true
timer.Create("sn_dc_holstertt", 0.30, 10, function()
uh_count = uh_count + 1

end)
timer.Simple(2, function()
uh_status = false
uh_count = 1
uh_vs = 1
end)
end)



end



if(CLIENT) then
INV2_OPEN = false
INV2_PANEL = false
INV2_ITEM_H = false




net.Receive("drp_advi2_open_inv_tcl", function(ply, res, test)
local inv = net.ReadTable() or {}
local max = net.ReadInt(32) or 0

local function InvSlotsbusy()
	local intc = 0
	for k,v in pairs(inv) do
		if(v) then
			intc = intc + 1
		end
	end
	return intc
end

local x, y = 64*9+25, 300
local Main = vgui.Create( "DFrame" )
Main:SetPos( ScrW() / 2 - x * 0.8, ScrH() +25)
Main:SetSize( x, y )
Main:SetTitle( "Inventory (" .. tonumber(InvSlotsbusy()) .. "/" .. max .. ")")
Main:SetVisible( true )
Main:SetDraggable( false )
Main:ShowCloseButton( true )
Main:SetSkin("DarkRP")
Main:MakePopup()
Main.Think = function(self)
	if(LocalPlayer():isArrested()) then
		self:Close()
	end
end
Main.OnClose = function()
	INV2_OPEN = false
	INV2_PANEL = false
	INV2_ITEM_H = false
end
Main.slots = {}
Main:MoveTo(ScrW() / 2 - x * 0.5, ScrH() - y, 0.35, 0, -1, function() 
local x,y = Main:GetPos()
Inv_Use = vgui.Create( "DFrame" )
Inv_Use:SetPos( x-128/1.5 - 25, y+25)
Inv_Use:SetSize(128 / 1.5, 128 / 1.5)
Inv_Use:SetTitle("")
Inv_Use:SetVisible( true )
Inv_Use:SetDraggable( false )
Inv_Use:ShowCloseButton( false )
Inv_Use:MakePopup()
Inv_Use.ov = false
Inv_Use.Think = function(self)
	if!(INV2_PANEL) then
		self:Remove()
	end
end

Inv_Use:Receiver( "invi", function(panel, tabl, dropped, ind, mx, my)
	if(dropped) then
		local inf
		local item = tabl[1]
		if(item.item) then
			inf = {class = item.item, id = item.idz}
		end
		if(Getinv2ItemInfo(inf.class)) then
			if!(Getinv2ItemInfo(inf.class).canuse(LocalPlayer(), inf.class)) then
				Inv_Use.ov = false
				Inv_Destroy.ov = false
					if(INV2_PANEL and INV2_PANEL:IsValid()) then
						if(INV2_PANEL.slots) then
							for k,v in pairs(INV2_PANEL.slots) do
								v.ov = false
							end
						end
					end
				return
			end
		end
		item:GetParent().item = false
		item:Remove()
		LocalPlayer():ConCommand("_drp_advi_useitem " .. inf.class .. " 1 "..inf.id)
		for k,v in pairs(Main.slots) do
			v.ov = false
			if(Inv_Use) then Inv_Use.ov = false end
		end
		Inv_Use.ov = false
		Inv_Destroy.ov = false
		if(INV2_PANEL and INV2_PANEL:IsValid()) then
			if(INV2_PANEL.slots) then
				for k,v in pairs(INV2_PANEL.slots) do
					v.ov = false
				end
			end
		end
	else
		
		Inv_Use.ov = true
	//	Inv_Destroy.ov = true
	end
end, {} )

Inv_Use.Paint = function(self)
	local x, y = self:GetSize()

	surface.SetDrawColor( 255, 255, 255, 255 )
	surface.SetMaterial( Material("advi2_icons/inv2_use.png")	)
	surface.DrawTexturedRect( 0, 0, x, y )
	if(self.ov) then
		surface.SetDrawColor(0, 150, 0, 150)
		draw.SimpleTextOutlined("Use", "DebugFixedSmall", 42,11, Color(1,255,1, 255), TEXT_ALIGN_CENTER, TEXT_ALIGN_CENTER, 1, Color(1,1,1))
	else
		surface.SetDrawColor(125, 125, 125, 25)
		draw.SimpleTextOutlined("Use", "DebugFixedSmall", 42,11, Color(255,255,255, 255), TEXT_ALIGN_CENTER, TEXT_ALIGN_CENTER, 1, Color(1,1,1))
	end
	surface.DrawOutlinedRect(0, 0, x, y)
end
--


//

Inv_Destroy = vgui.Create( "DFrame" )
Inv_Destroy:SetPos( x-128/1.5 - 25, y+205)
Inv_Destroy:SetSize(128 / 1.5, 128 / 1.5)
Inv_Destroy:SetTitle("")
Inv_Destroy:SetVisible( true )
Inv_Destroy:SetDraggable( false )
Inv_Destroy:ShowCloseButton( false )
Inv_Destroy:MakePopup()
Inv_Destroy.ov = false
Inv_Destroy.Think = function(self)
	if!(INV2_PANEL) then
		self:Remove()
	end
end

Inv_Destroy:Receiver( "invi", function(panel, tabl, dropped, ind, mx, my)
	if(dropped) then
		local inf
		local item = tabl[1]
		if(item.item) then
			inf = {class = item.item, id = item.idz}
		end
		item:GetParent().item = false
		item:Remove()
		LocalPlayer():ConCommand("_drp_advi_destritem " .. inf.class .. " "..inf.id)

		for k,v in pairs(Main.slots) do
			v.ov = false
			if(Inv_Use) then Inv_Use.ov = false end
		end

	
		Inv_Use.ov = false
		Inv_Destroy.ov = false
		if(INV2_PANEL and INV2_PANEL:IsValid()) then
			if(INV2_PANEL.slots) then
				for k,v in pairs(INV2_PANEL.slots) do
					v.ov = false
				end
			end
		end
	else
		
		//Inv_Use.ov = true
		Inv_Destroy.ov = true
	end
end, {} )

Inv_Destroy.Paint = function(self)
	local x, y = self:GetSize()

	surface.SetDrawColor( 255, 255, 255, 255 )
	surface.SetMaterial( Material("advi2_icons/inv2_destroy.png")	)
	surface.DrawTexturedRect( 0, 0, x, y )
	if(self.ov) then
		surface.SetDrawColor(0, 150, 0, 150)
		draw.SimpleTextOutlined("Destroy", "DebugFixedSmall", 41,11, Color(255,1,1, 255), TEXT_ALIGN_CENTER, TEXT_ALIGN_CENTER, 1, Color(1,1,1))
	else
		surface.SetDrawColor(125, 125, 125, 25)
		draw.SimpleTextOutlined("Destroy", "DebugFixedSmall", 41,11, Color(255,1,1, 255), TEXT_ALIGN_CENTER, TEXT_ALIGN_CENTER, 1, Color(1,1,1))
	end
	surface.DrawOutlinedRect(0, 0, x, y)
end
--


Inv_Drop = vgui.Create( "DFrame" )
Inv_Drop:SetPos( x-128/1.5 - 25, y+115)
Inv_Drop:SetSize(128 / 1.5, 128 / 1.5)
Inv_Drop:SetTitle("")
Inv_Drop:SetVisible( true )
Inv_Drop:SetDraggable( false )
Inv_Drop:ShowCloseButton( false )
Inv_Drop:MakePopup()
Inv_Drop.ov = false
Inv_Drop.Think = function(self)
	if!(INV2_PANEL) then
		self:Remove()
	end
end

Inv_Drop:Receiver( "invi", function(panel, tabl, dropped, ind, mx, my)
	if(dropped) then
		local inf
		local item = tabl[1]
		if(item.item) then
			inf = {class = item.item, id = item.idz}
		end
		if(Getinv2ItemInfo(inf.class)) then
			if!(Getinv2ItemInfo(inf.class).candrop(LocalPlayer(), inf.class)) then
				Inv_Use.ov = false
				Inv_Destroy.ov = false
					if(INV2_PANEL and INV2_PANEL:IsValid()) then
						if(INV2_PANEL.slots) then
							for k,v in pairs(INV2_PANEL.slots) do
								v.ov = false
							end
						end
					end
				return
			end
		end
		item:GetParent().item = false
		item:Remove()

		LocalPlayer():ConCommand("_drp_advi_dropitem " .. inf.class .. " "..inf.id)

		for k,v in pairs(Main.slots) do
			v.ov = false
			if(Inv_Use) then Inv_Use.ov = false end
		end

	
		Inv_Use.ov = false
		Inv_Destroy.ov = false
		Inv_Drop.ov = false
		if(INV2_PANEL and INV2_PANEL:IsValid()) then
			if(INV2_PANEL.slots) then
				for k,v in pairs(INV2_PANEL.slots) do
					v.ov = false
				end
			end
		end
	else
		
		//Inv_Use.ov = true
		Inv_Drop.ov = true
	end
end, {} )

Inv_Drop.Paint = function(self)
	local x, y = self:GetSize()

	surface.SetDrawColor( 255, 255, 255, 255 )
	surface.SetMaterial( Material("advi2_icons/inv2_drop.png")	)
	surface.DrawTexturedRect( 0, 0, x, y )
	if(self.ov) then
		surface.SetDrawColor(0, 150, 0, 150)
		draw.SimpleTextOutlined("Drop", "DebugFixedSmall", 41,11, Color(255,255,1, 255), TEXT_ALIGN_CENTER, TEXT_ALIGN_CENTER, 1, Color(1,1,1))
	else
		surface.SetDrawColor(125, 125, 125, 25)
		draw.SimpleTextOutlined("Drop", "DebugFixedSmall", 41,11, Color(255,255,1, 255), TEXT_ALIGN_CENTER, TEXT_ALIGN_CENTER, 1, Color(1,1,1))
	end
	surface.DrawOutlinedRect(0, 0, x, y)
end

end)
//





local iList = vgui.Create( "DPanelList", Main )
iList:SetPos( 1,25 )
iList:SetSize( Main:GetWide()-2, Main:GetTall()-25 )
iList:SetSpacing( 1 ) 
iList:EnableHorizontal( true ) 
iList:EnableVerticalScrollbar( true ) 
iList.Paint = function(self)
surface.SetDrawColor(55,55,55,255)
surface.DrawRect(0,0, self:GetWide(), self:GetTall())
end

for i = 1, max do
	local item
	if(inv[i]) then
		item = inv[i]
	else
		item = false
	end
	local slot = vgui.Create( "DPropertySheet" )
	slot:SetSize( 64, 64 )
	slot.item = item
	slot:SetSkin("DarkRP")
	slot.ov = false
	slot.count = i
	slot:Receiver( "invi", function(panel, tabl, dropped, ind, mx, my)
		if(dropped) then
			local is = tabl[1]
			if(slot.item) then
				for k,v in pairs(Main.slots) do
					v.ov = false
				end
				if(Inv_Use) then Inv_Use.ov = false end
				if(Inv_Destroy) then Inv_Destroy.ov = false end
				return
			end
			net.Start("drp_advi2_item_changed")
			net.WriteTable({from = is:GetParent().count, to = slot.count})
			net.SendToServer()
			slot.item = is:GetParent().item
			is:GetParent().item = false
			is:SetParent(slot)

			for k,v in pairs(Main.slots) do
				v.ov = false
				if(Inv_Use) then Inv_Use.ov = false end
				if(Inv_Destroy) then Inv_Destroy.ov = false end
			end
		else
			for k,v in pairs(Main.slots) do
				if!(v.item) then
					v.ov = true
				end
			end

		end
	end, {} )
	slot.PaintOver = function(self)
	if(LocalPlayer():KeyDown(IN_SPEED)) then
		draw.SimpleTextOutlined(i, "DebugFixed", 10,10, Color(255,255,255, 5), TEXT_ALIGN_CENTER, TEXT_ALIGN_CENTER, 0, Color(1,1,1))
	end
		if(self.ov) then
			surface.SetDrawColor(0,255,0, 15)
			surface.DrawOutlinedRect(2, 2, self:GetWide() - 4, self:GetTall() - 4)
		end	
	end
	if(item) then
		local Icon = vgui.Create( "SpawnIcon" , slot ) -- SpawnIcon
		Icon.OnCursorEntered = function(self)
   	 		return false
		end
		Icon:SetPos(0,0)
		Icon.item = item
		Icon.idz = tonumber(i)
		Icon:SetSize(64,64)
		Icon:SetModel(Getinv2ItemInfo(item)["model"])
		Icon:Droppable( "invi" )
		Icon:SetToolTip(nil)
		Icon.hover = false
		Icon.OnCursorEntered = function(self) INV2_ITEM_H = item self.hover = true end
		Icon.OnCursorExited = function(self)  INV2_ITEM_H = false self.hover = false end
		Icon.PaintOver = function(self)
			if(self.hover) then
			surface.SetDrawColor(125,125,125, 25)
			surface.DrawOutlinedRect(2, 2, self:GetWide()-4, self:GetTall()-4 )
			end
		end
	end

	Main.slots[i] = slot
	iList:AddItem(slot)
end


INV2_PANEL = Main
end)		

hook.Add("Think", "drp_advi_invbc", function()
	if(LocalPlayer() and LocalPlayer():IsValid() and LocalPlayer():IsPlayer() and LocalPlayer():Alive()) then
		if(LocalPlayer():KeyDown(IN_SPEED) ) then
			if(input.IsKeyDown(KEY_B)) then
				if(INV2_OPEN == false) then
					INV2_OPEN = true
					if(LocalPlayer():isArrested()) then INV2_OPEN = false return  end
					net.Start("drp_advi2_open_inv_fcl")
					net.SendToServer()
					return
				end
			end
		end
	end
end)

hook.Add("HUDPaint", "drp_advi2_draw_1", function()
	if!(INV2_OPEN) then
		if(LocalPlayer() and LocalPlayer():IsValid() and LocalPlayer():IsPlayer() and LocalPlayer():Alive()) then
			if(SH_ADVANCED_INV_V2["Settings"].inventory_hud) then
				local hudi = SH_ADVANCED_INV_V2["Settings"].inventory_hud
				if(hudi.enabled) then
					local x = ScrW() / 2
					local y = ScrH() - 19.99
					if(hudi.position) then
						local pos = tostring(hudi.position)
						if(pos == "bottom") then
							x = ScrW() / 2
							y = ScrH() - 19.99
						end
						if(pos == "top") then
							x = ScrW() / 2
							y = 0
						end
					end
					surface.SetDrawColor(Color(50, 50, 50, 230))
					surface.DrawRect(x - 260 / 2, y, 260, 20)
					surface.SetDrawColor(Color(1, 1, 1, 255))
					surface.DrawOutlinedRect(x - 260 / 2, y, 260, 20)
					draw.SimpleTextOutlined("Press SHIFT-B to Open Your Inventory", "DebugFixedSmall", x, y+10, Color(255,255,255), TEXT_ALIGN_CENTER, TEXT_ALIGN_CENTER, 1, Color(1,1,1))
				end
			end
		end
	else

		if(LocalPlayer() and LocalPlayer():IsValid() and LocalPlayer():IsPlayer() and LocalPlayer():Alive() and INV2_ITEM_H) then
			local x, y = INV2_PANEL:GetPos()
			x = ScrW() / 2
			y = y - 100
			if(isValidInvItem(INV2_ITEM_H)) then
				local info = Getinv2ItemInfo(INV2_ITEM_H)
				draw.SimpleTextOutlined(info.name, "sn_advi2font_4", x, y, info.color1, TEXT_ALIGN_CENTER, TEXT_ALIGN_CENTER, 1, Color(1,1,1))
				draw.SimpleTextOutlined(info.desc, "sn_advi2font_3", x, y+50, info.color2, TEXT_ALIGN_CENTER, TEXT_ALIGN_CENTER, 1, Color(1,1,1))
			end
		end
	end
end)


local function AwesomePanelsExists()
	for k,v in pairs(DarkRP.getF4MenuPanel():GetTable().Items) do
		if(v.Panel.sn) then
			return true
		end
	end
	return false
end


surface.CreateFont( "sn_advi2font_1", {
	font = "Arial",
	size = 25,
	weight = 500,
	blursize = 0,
	scanlines = 0,
	antialias = true,
	underline = false,
	italic = false,
	strikeout = false,
	symbol = false,
	rotary = false,
	shadow = true,
	additive = false,
	outline = true,
} )


surface.CreateFont( "sn_advi2font_2", {
	font = "Arial Black",
	size = 18,
	weight = 500,
	blursize = 0,
	scanlines = 0,
	antialias = true,
	underline = false,
	italic = false,
	strikeout = false,
	symbol = false,
	rotary = false,
	shadow = true,
	additive = false,
	outline = false,
} )

surface.CreateFont( "sn_advi2font_3", {
	font = "Arial",
	size = 16,
	weight = 300,
	blursize = 0,
	scanlines = 0,
	antialias = true,
	underline = false,
	italic = false,
	strikeout = false,
	symbol = false,
	rotary = false,
	shadow = false,
	additive = false,
	outline = false,
} )

surface.CreateFont( "sn_advi2font_4", {
	font = "Arial",
	size = 26,
	weight = 300,
	blursize = 0,
	scanlines = 0,
	antialias = true,
	underline = false,
	italic = false,
	strikeout = false,
	symbol = false,
	rotary = false,
	shadow = false,
	additive = false,
	outline = false,
} )



function NewAdviShop()
local SN_NewShop_MAIN = vgui.Create( "DPropertySheet" )
SN_NewShop_MAIN.sn = true
SN_NewShop_MAIN:SetSkin("Default")
local shop_cats = {}

for c,d in pairs(SH_ADVANCED_INV_V2["Item_Categories"]) do
shop_cats[c] = vgui.Create( "DPropertySheet" )
shop_cats[c]:SetSkin("DarkRP")

local ShopList = vgui.Create( "DPanelList" , shop_cats[c])
ShopList:SetPos(0, 0)
ShopList:SetSize(shop_cats[c]:GetParent():GetWide()-232, shop_cats[c]:GetParent():GetTall()-290)
ShopList:SetSpacing( 1 ) 
ShopList:EnableHorizontal( false ) 
ShopList:EnableVerticalScrollbar( true ) 
ShopList.Paint = function(self)
surface.SetDrawColor(25,25,25,255)
surface.DrawRect(0,0, self:GetWide(), self:GetTall())
end
local curcat = tostring(d.name)

for k,v in pairs(SH_ADVANCED_INV_V2["Items"]) do
if(v.buy["BuyAble"]) then
if(tostring(curcat) == tostring(v.category)) then
local imain = vgui.Create( "DPropertySheet" )
imain:SetSize(ShopList:GetWide(), 150)
imain:SetSkin("DarkRP")
imain.Paint = function(self)
local x, y = self:GetSize()
surface.SetDrawColor(1, 1, 1, 255)
surface.DrawRect(0, 0, x, y)
surface.SetDrawColor( 25, 25, 25, 255 );
surface.SetTexture( surface.GetTextureID( "gui/gradient_up" ) );
surface.DrawTexturedRect(0, 0, x, y);	

surface.SetDrawColor(125, 125, 125, 255)
surface.DrawOutlinedRect(0, 0, x, y)


draw.RoundedBoxEx(16, 10, 48, 64*2, 64*1.3, Color(1, 1, 1, 255), true, false, false, true)
draw.RoundedBoxEx(16, 14, 52, 64*2, 64*1.3, Color(1, 1, 1, 255), true, false, false, true)
draw.RoundedBoxEx(16, 12, 50, 64*2, 64*1.3, Color(45, 45, 45, 255), true, false, false, true)




draw.SimpleTextOutlined(v.name or "?", "sn_advi2font_1", 12, 8, v.color1, TEXT_ALIGN_LEFT, TEXT_ALIGN_LEFT, 1, Color(1,1,1))

//Misc Info
draw.RoundedBoxEx(8, 350, 115, 450, 35, Color(125, 125, 125, 25, 255), true, true, false, false)

local limit = tonumber(math.floor(v.max)) or 0
if(limit > 0) then
	draw.SimpleTextOutlined("Limit: " .. v.max, "sn_advi2font_3", 350+5+100*3.4, 115+10, Color(255,1,1), TEXT_ALIGN_LEFT, TEXT_ALIGN_LEFT, 0.5, Color(1,1,1))
else
	draw.SimpleTextOutlined("Limit: Unlimited", "sn_advi2font_3", 350+5+100*3.4, 115+10, Color(255,255,255), TEXT_ALIGN_LEFT, TEXT_ALIGN_LEFT, 0.5, Color(1,1,1))
end
local price = tonumber(math.floor(v.buy["Price_Each"])) or 0
price = formatNumber(price)

draw.SimpleTextOutlined("Price: $" ..price, "sn_advi2font_3", 350+5+100*0, 115+10, Color(1,255,1), TEXT_ALIGN_LEFT, TEXT_ALIGN_LEFT, 0.5, Color(1,1,1))

local shipment = v.buy["Shipments"]["Enable"]
if(shipment) then
local shipment_size = v.buy["Shipments"]["ShipmentSize"] 
local shipment_discount = v.buy["Shipments"]["ShipmentDiscount"] 

draw.SimpleTextOutlined("Shiment Size:"..shipment_size.." ("..shipment_discount.."% Discount)", "sn_advi2font_3", 350+5+100*1.2, 115+10, Color(255,255,255), TEXT_ALIGN_LEFT, TEXT_ALIGN_LEFT, 0, Color(255,255,255))
end



end

local Preview = vgui.Create( "DModelPanel" , imain)
Preview:SetModel( v.model )
Preview:SetPos(400,1)
Preview:SetSize( 300, 100 )
Preview:SetFOV( 32 )
Preview.ov = 0
Preview:SetCamPos( Vector( 0, 100, 0 ) )
Preview:SetLookAt( Vector( 0, 0, 5 ) )
function Preview:LayoutEntity( Entity )
	if(Preview.ov == 0) then
		Entity:SetAngles(Entity:GetAngles() + Angle(0, 0.2, 0))
	else
		Entity:SetAngles(Entity:GetAngles() + Angle(0, Preview.ov, 0))
	end
end

Preview.PaintOver = function(self)
//draw.SimpleTextOutlined("", "DebugFixedSmall", 60,5, Color(255,255,255), TEXT_ALIGN_LEFT, TEXT_ALIGN_LEFT, 0.5, Color(1,1,1))
end


Preview.OnMouseWheeled = function(self, s)
return
end


local MDL_Left = vgui.Create( "DButton", Preview )
MDL_Left:SetPos( 1, 1 )
MDL_Left:SetText( "" )
MDL_Left:SetSize( 25, 25 )
MDL_Left.OnMousePressed = function()
	Preview.ov = 1
end
MDL_Left.OnMouseReleased = function()
	Preview.ov = 0
end
MDL_Left.OnCursorExited = function(self) Preview.ov = 0 end
MDL_Left.Paint = function(self)
local x,y = self:GetSize()
draw.RoundedBoxEx(4*3, 1, 1, x, y, Color(125, 125, 125, 25, 255), true, false, true, false)
surface.SetDrawColor( 255, 255, 255, 255 )
surface.SetMaterial( Material("icon16/arrow_left.png") )
surface.DrawTexturedRect( 5, 5, 16, 16 )
end
--

local MDL_Right = vgui.Create( "DButton", Preview )
MDL_Right:SetPos( Preview:GetWide() - 30, 1 )
MDL_Right:SetText( "" )
MDL_Right:SetSize( 25, 25 )
MDL_Right.OnMousePressed = function()
	Preview.ov = -1
end
MDL_Right.OnMouseReleased = function()
	Preview.ov = 0
end
MDL_Right.OnCursorExited = function(self) Preview.ov = 0 end
MDL_Right.Paint = function(self)
local x,y = self:GetSize()
draw.RoundedBoxEx(4*3, 1, 1, x, y, Color(125, 125, 125, 25, 255), false, true, false, true)
surface.SetDrawColor( 255, 255, 255, 255 )
surface.SetMaterial( Material("icon16/arrow_right.png") )
surface.DrawTexturedRect( 5, 5, 16, 16 )
end



local BuyButton = vgui.Create( "DButton", imain )
BuyButton:SetPos( imain:GetWide() - 220, 50)
BuyButton:SetText( "Purchase" )
BuyButton:SetSize( 200, 50 )

BuyButton:SetFont("sn_advi2font_1")
BuyButton:SetSkin("DarkRP")
BuyButton.Paint = function(self)
local x,y = self:GetSize()

if(self:IsHovered()) then
draw.RoundedBoxEx(4, 0, 0, x, y, Color(255, 255, 255, 255), true, true, true, true)
draw.RoundedBoxEx(4, 4, 4, x, y, Color(255, 255, 255, 255), true, true, true, true)
else
draw.RoundedBoxEx(4, 0, 0, x, y, Color(255, 255, 255, 25), true, true, true, true)
draw.RoundedBoxEx(4, 4, 4, x, y, Color(255, 255, 255, 25), true, true, true, true)
end
draw.RoundedBoxEx(4, 2, 2, x-4, y-4, Color(55, 55, 55, 255), true, true, true, true)
end


BuyButton.OnMousePressed = function(self)
if(self:GetDisabled()) then
	return
end
local MenuButtonOptions = DermaMenu()	



MenuButtonOptions.Paint = function(self)
local x,y = self:GetSize()
draw.RoundedBoxEx(6, 0, 0, x, y, Color(25,25,25, 252), true, true, false, false)
end

local am = MenuButtonOptions:AddOption("Buy (How Many?)", function()
end)
am:SetIcon( "icon16/bricks.png" )
local amz = am:AddSubMenu()
amz.Paint = function(self)
local x,y = self:GetSize()
draw.RoundedBoxEx(4*3, 0, 0, x, y, Color(36,18,0, 200), true, true, false, false)
end

for r,c in pairs(v.buy["Buy_Sizes"]) do
	amz:AddOption( c.."x ($"..formatNumber(tonumber(math.floor(v.buy["Price_Each"]*c)))..")", function()
		LocalPlayer():ConCommand("_drp_advi_buyfs " ..v.class.." "..c) 
	end):SetIcon( "icon16/brick_go.png" )
end

if(v.buy["Shipments"]["Enable"]) then
local disc = v.buy["Shipments"]["ShipmentDiscount"]
local shipsize = v.buy["Shipments"]["ShipmentSize"]
local shipprice = v.buy["Price_Each"] * shipsize
shipprice = math.floor(shipprice)
local totaldisc = tonumber(math.floor(shipprice / disc))
local discprice = formatNumber(tonumber(math.floor(shipprice-totaldisc)))



local ship = MenuButtonOptions:AddOption("Buy Shipment("..shipsize.."x) $"..discprice .. " ("..disc.."% Shipment Discount)", function()
	LocalPlayer():ConCommand("_drp_advi_buyfs " ..v.class.." ".."0") 
end)
ship:SetIcon( "icon16/box.png" )
end

local close = MenuButtonOptions:AddOption("Close", function()
end)
close:SetIcon( "icon16/cross.png" )




MenuButtonOptions:Open() 
local px, py = MenuButtonOptions:GetPos()
MenuButtonOptions:SetPos(px, py+30)
end




local Item_Desc = vgui.Create( "DTextEntry", imain )	-- create the form as a child of frame
Item_Desc:SetPos( 160, 35 )
Item_Desc:SetSize( 200, 109)

Item_Desc:SetValue("" .. v.desc )
Item_Desc:SetMultiline(true)
Item_Desc:SetFont("sn_advi2font_2")
Item_Desc:SetTextColor(v.color2)
Item_Desc:SetEditable(false)
Item_Desc:SetDrawBackground(false)
Item_Desc.OnEnter = function( self )
	return
end
Item_Desc.PaintOver = function(self)
local x, y = self:GetSize()


end


local icon = vgui.Create( "SpawnIcon", imain )
icon.OnCursorEntered = function(self)
   	 return false
end
icon:SetPos(32,55)
icon:SetSize( 64, 64 )
icon:SetModel( v.model ) 
icon:SetToolTip(nil)
icon.PaintOver = function(self)
end







ShopList:AddItem(imain)
end
end
end


SN_NewShop_MAIN:AddSheet( d.name, shop_cats[c], d.icon, false, false, d.name)
end







return SN_NewShop_MAIN
end




hook.Add("F4MenuTabs", "F4MenuTabs_Magicadvi2", function()
DarkRP.removeF4MenuTab("Ammo")
DarkRP.removeF4MenuTab("Food")
DarkRP.removeF4MenuTab("Weapons")
DarkRP.removeF4MenuTab("Shipments")

if!(AwesomePanelsExists()) then
DarkRP.addF4MenuTab("Shop", NewAdviShop())
else
DarkRP.removeF4MenuTab("Shop")
DarkRP.addF4MenuTab("Shop", NewAdviShop())
end
end)

end


if(SERVER) then

concommand.Add("_drp_advi_buyfs", function(ply, c, a)
	if(ply and ply:IsValid() and ply:IsPlayer() and ply:Alive()) then
		if(a and a[1] and a[2] and #a == 2) then
			if(tostring(a[1])) then
				if(tonumber(a[2])) then
					local item = tostring(a[1])
					local form = tonumber(a[2])
					if(isValidInvItem(item)) then
						if!(Getinv2ItemInfo(item).buy["BuyAble"]) then
							return ""
						end
					end
					if!(form == 0) then
						if(isValidInvItem(item)) then
							local info = Getinv2ItemInfo(item)
							if(#info.teams == 0) then
								//
							else
								if!(table.HasValue(info.teams, ply:Team())) then
									ply:ChatPrint("Your job can't buy this item.")
								return ""
							end
							end
							if(table.HasValue(info.buy.Buy_Sizes, form)) then
								if(info.customcheck(ply)) then
									local price = tonumber(math.floor(info.buy.Price_Each * form))
									if(ply:canAfford(price)) then
										if!(ply:AdviCanHaveMoreOf(item, form)) then
											ply:ChatPrint("You can't own more items of this kind.")
											return ""
										end
										ply:addMoney(-price)
										if(ply:HasInv2Space(form)) then
											ply:AddinvItem(item, form, true)
											ply:SaveAdvinv()
											ply:ChatPrint("You've purchased "..form.."x "..info["name"].. " For $"..formatNumber(price))
											return
										end
									end
								end
							end
						end
					else
						if(isValidInvItem(item)) then
							local info = Getinv2ItemInfo(item)
							if(#info.teams == 0) then
								//
							else
								if!(table.HasValue(info.teams, ply:Team())) then
									ply:ChatPrint("Your job can't buy this item.")
								return ""
							end
							end

							if(info.buy.Shipments.Enable) then
								local size = tonumber(math.floor(info.buy.Shipments.ShipmentSize))
								local disc = info.buy["Shipments"]["ShipmentDiscount"]
								local shipsize = info.buy["Shipments"]["ShipmentSize"]
								local shipprice = info.buy["Price_Each"] * shipsize
								shipprice = math.floor(shipprice)
								local totaldisc = tonumber(math.floor(shipprice / disc))
								local discprice = tonumber(math.floor(shipprice-totaldisc))
								if(ply:canAfford(discprice)) then
									ply:addMoney(-discprice)

									local trace = {}
									trace.start = ply:EyePos()
									trace.endpos = trace.start + ply:GetAimVector() * 85
									trace.filter = ply

									local tr = util.TraceLine(trace)
	
									local crate = ents.Create("advi2_shipment")
									crate.SID = ply.SID
									crate:SetPos(tr.HitPos)
									crate.nodupe = true
									crate.dt.weapon = Advi2ClassToID(item)
									crate.dt.count = 10
									crate:SetModel(SH_ADVANCED_INV_V2["Settings"].shipment_model)
									crate:Spawn()
									crate:Activate()

									ply:ChatPrint("You've purchased a Shipment of "..info["name"].. " For $"..formatNumber(discprice))
									return
								end
							end
						end
					end
				end
			end
		end
	end
end)


end
