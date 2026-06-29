-- Sirius script for TLK Prison (Delta Executor Android) - Restored Stable Movement & Obsidian UI Icons
local repo = "https://raw.githubusercontent.com/deividcomsono/Obsidian/main/"
local Library = loadstring(game:HttpGet(repo .. "Library.lua"))()

Library.ForceCheckbox = false
Library.ShowToggleFrameInKeybinds = true

local Window = Library:CreateWindow({
	Title = "Sirius",              
	Footer = "Sirius | BETA.",     
	NotifySide = "Right",
	ShowCustomCursor = true,
})

-- Создаём вкладки с оригинальными иконками Obsidian UI (Шестерёнка на Main)
local Tabs = {
	Main = Window:AddTab("Main", "settings"),
	Player = Window:AddTab("Player", "user"),
	Helper = Window:AddTab("Helper", "shield"),
	Combat = Window:AddTab("Combat", "swords"),
	TeleportMap = Window:AddTab("Teleport map", "map"),
	TeleportWeapons = Window:AddTab("Teleport weapons", "shield-alert")
}

local player = game:GetService("Players").LocalPlayer
local runService = game:GetService("RunService")
local userInputService = game:GetService("UserInputService")
local camera = workspace.CurrentCamera

-- Глобальная проверка персонажа
local function isCharacterReady()
	local char = player.Character
	if not char then return false end
	local hum = char:FindFirstChildOfClass("Humanoid")
	local hrp = char:FindFirstChild("HumanoidRootPart")
	if not hum or not hrp or hum.Health <= 0 then return false end
	return true
end

-- Исходные переменные
local safeSpeed = 0.29 
local auraRadius = 14.8 
local hbSize = 13 
local fastAttackEnabled = false
local hitboxEnabled = false
local hitboxSize = 12
local isFreecamEnabled = false
local camPart = nil
local loopConnectionCam = nil
local touchConnectionCam = nil
local freeCamSpeed = 2.0
local cameraYaw = 0
local cameraPitch = 0

-- Переменные для триггерботов
local triggerBotEnabled = false
local triggerBot2Enabled = false 
local triggerBotRadius = 8.3

-- Переменные для Walk Sky, Anti-Kick и God Mode
local walkSkyEnabled = false
local lastPlatformPos = Vector3.new()
local antiKickEnabled = false
local antiKickV2Enabled = false
local shouldGodModeTeleport = false


-- === ВАКЛАДКА MAIN ===
local MainLeftGroupBox = Tabs.Main:AddLeftGroupbox("Credits")
MainLeftGroupBox:AddLabel("Owner script - Now_master(@w1eskiil)\nScr67kid(@ppappaapu)", true)


-- === ВКЛАДКА TELEPORT MAP ===
local TeleportMapGroupBox = Tabs.TeleportMap:AddLeftGroupbox("Teleport map")
TeleportMapGroupBox:AddLabel("So far, the tab is empty.")


-- === ВКЛАДКА TELEPORT WEAPONS ===
local TeleportWeaponsGroupBox = Tabs.TeleportWeapons:AddLeftGroupbox("Teleport weapons")
TeleportWeaponsGroupBox:AddLabel("So far, the tab is empty.")


-- === ВКЛАДКА PLAYER ===
local TpWalkGroupBox = Tabs.Player:AddLeftGroupbox("tpwalk")
local GodModeGroupBox = Tabs.Player:AddLeftGroupbox("God mode person")
local WalkSkyGroupBox = Tabs.Player:AddRightGroupbox("Walking the sky")
local AntiKickGroupBox = Tabs.Player:AddLeftGroupbox("Anti kick")
local FreecamGroupBox = Tabs.Player:AddRightGroupbox("Freecam")

local tpWalkEnabled = false
local tpWalkSpeed = 16 
local TpWalkButton

-- Логика отслеживания смерти и спавна для God Mode
local function setupDeathResetFix(character)
	local humanoid = character:WaitForChild("Humanoid", 5)
	if humanoid then
		humanoid.Died:Connect(function()
			if tpWalkEnabled then
				tpWalkEnabled = false
				if TpWalkButton then TpWalkButton:SetText("tpwalk: Off") end
			end
		end)
	end
	
	if shouldGodModeTeleport then
		local hrp = character:WaitForChild("HumanoidRootPart", 10)
		if hrp then
			task.wait(0.15)
			hrp.CFrame = CFrame.new(255.94, 3.15, -95.94)
			shouldGodModeTeleport = false
		end
	end
end

if player.Character then setupDeathResetFix(player.Character) end
player.CharacterAdded:Connect(setupDeathResetFix)

-- КНОПКА GOD MODE
GodModeGroupBox:AddButton({
	Text = "God Mode",
	Func = function()
		if isCharacterReady() then
			shouldGodModeTeleport = true
			player.Character:FindFirstChildOfClass("Humanoid").Health = 0
		end
	end
})

-- ИЗНАЧАЛЬНЫЙ СТАБИЛЬНЫЙ TPWALK (Без влияния на камеру)
task.spawn(function()
	while task.wait() do
		if Library.Unloaded then break end
		if isCharacterReady() and tpWalkEnabled then
			pcall(function()
				local character = player.Character
				local humanoid = character:FindFirstChildOfClass("Humanoid")
				if humanoid and humanoid.MoveDirection.Magnitude > 0 then
					character:TranslateBy(humanoid.MoveDirection * (tpWalkSpeed * 0.005))
				end
			end)
		end
	end
end)

TpWalkButton = TpWalkGroupBox:AddButton({
	Text = "tpwalk: Off",
	Func = function()
		if not isCharacterReady() then return end
		tpWalkEnabled = not tpWalkEnabled
		TpWalkButton:SetText(tpWalkEnabled and "tpwalk: On" or "tpwalk: Off")
	end
})

TpWalkGroupBox:AddSlider("TpWalkSlider", {
	Text = "Speed Level", Default = 16, Min = 0, Max = 100, Rounding = 0, Compact = false,
	Callback = function(Value) tpWalkSpeed = Value end
})

-- ANTI-KICK BYPASS V1
task.spawn(function()
	while task.wait(0.5) do
		if Library.Unloaded then break end
		if antiKickEnabled and player then
			pcall(function()
				if player:FindFirstChild("Kick") or player:FindFirstChild("kick") then
					local mt = getrawmetatable(game)
					if mt and setreadonly then
						setreadonly(mt, false)
						local old = mt.__index
						mt.__index = function(t, k)
							if antiKickEnabled and t == player and (k == "Kick" or k == "kick") then
								return function() return nil end
							end
							return old(t, k)
						end
						setreadonly(mt, true)
					end
				end
			end)
		end
	end
end)

local AntiKickButton
AntiKickButton = AntiKickGroupBox:AddButton({
	Text = "anti kick Bypass: Off",
	Func = function()
		antiKickEnabled = not antiKickEnabled
		AntiKickButton:SetText(antiKickEnabled and "anti kick Bypass: On" or "anti kick Bypass: Off")
	end
})

-- ИЗНАЧАЛЬНЫЙ РОДНОЙ WALK SKY LOGIC
WalkSkyGroupBox:AddToggle("WalkSkyToggle", {
	Text = "Enable Walk Sky", Default = false,
	Callback = function(Value) walkSkyEnabled = Value end
})

runService.Heartbeat:Connect(function()
	if not walkSkyEnabled or not isCharacterReady() then return end
	pcall(function()
		local character = player.Character
		local hrp = character.HumanoidRootPart
		local humanoid = character:FindFirstChildOfClass("Humanoid")
		local currentPos = hrp.Position
		
		if (currentPos - lastPlatformPos).Magnitude > 2.5 then
			lastPlatformPos = currentPos
			
			local skyPart = Instance.new("Part")
			skyPart.Shape = Enum.PartType.Block
			skyPart.Size = Vector3.new(5, 0.5, 5)
			skyPart.Color = Color3.fromRGB(0, 170, 255)
			skyPart.Transparency = 0.7
			skyPart.Anchored = true
			skyPart.CanCollide = true
			
			if humanoid and humanoid.MoveDirection.Magnitude > 0 then
				skyPart.CFrame = CFrame.new(currentPos.X, currentPos.Y - 3.6, currentPos.Z) * CFrame.Angles(math.rad(-8), 0, 0)
			else
				skyPart.CFrame = CFrame.new(currentPos.X, currentPos.Y - 4.2, currentPos.Z)
			end
			
			skyPart.Parent = workspace
			task.delay(0.8, function() if skyPart then skyPart:Destroy() end end)
		end
	end)
end)

-- FREECAM LOGIC
local function enableFreecam()
    if not isCharacterReady() then return end
    player.Character.HumanoidRootPart.Anchored = true
    camPart = Instance.new("Part")
    camPart.Size = Vector3.new(1, 1, 1)
    camPart.Transparency = 1
    camPart.CanCollide = false
    camPart.Anchored = true
    camPart.CFrame = camera.CFrame
    camPart.Parent = workspace
    
    local lookVector = camera.CFrame.LookVector
    cameraYaw = math.atan2(-lookVector.X, -lookVector.Z)
    cameraPitch = math.asin(lookVector.Y)
    camera.CameraType = Enum.CameraType.Scriptable
    
    touchConnectionCam = userInputService.TouchMoved:Connect(function(touch, gameProcessed)
        if gameProcessed then return end
        local delta = touch.Delta
        cameraYaw = cameraYaw - delta.X * 0.008
        cameraPitch = math.clamp(cameraPitch - delta.Y * 0.008, -math.rad(85), math.rad(85))
    end)
    
    loopConnectionCam = runService.RenderStepped:Connect(function()
        if camPart and camera.CameraType == Enum.CameraType.Scriptable and isCharacterReady() then
            camPart.CFrame = CFrame.new(camPart.CFrame.Position) * CFrame.Angles(0, cameraYaw, 0) * CFrame.Angles(cameraPitch, 0, 0)
            camera.CFrame = camPart.CFrame
            local humanoid = player.Character:FindFirstChildOfClass("Humanoid")
            if humanoid and humanoid.MoveDirection.Magnitude > 0 then
                camPart.Position = camPart.Position + (camera.CFrame.LookVector * (humanoid.MoveDirection.Z * -freeCamSpeed)) + (camera.CFrame.RightVector * (humanoid.MoveDirection.X * freeCamSpeed))
            end
        end
    end)
end

local function disableFreecam()
    if loopConnectionCam then loopConnectionCam:Disconnect() loopConnectionCam = nil end
    if touchConnectionCam then touchConnectionCam:Disconnect() touchConnectionCam = nil end
    if camPart then camPart:Destroy() camPart = nil end
    
    if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then 
		local hrp = player.Character.HumanoidRootPart
		hrp.Anchored = false 
		hrp.Velocity = Vector3.new(0, 0, 0)
		hrp.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
	end
	
    camera.CameraType = Enum.CameraType.Custom
    if player.Character and player.Character:FindFirstChild("Humanoid") then 
		local hum = player.Character.Humanoid
		camera.CameraSubject = hum
		hum:ChangeState(Enum.HumanoidStateType.GettingUp)
		hum:ChangeState(Enum.HumanoidStateType.Running)
	end
end

local FreecamButton
FreecamButton = FreecamGroupBox:AddButton({
	Text = "Enable Freecam: Off",
	Func = function()
		isFreecamEnabled = not isFreecamEnabled
		FreecamButton:SetText(isFreecamEnabled and "Enable Freecam: On" or "Enable Freecam: Off")
		if isFreecamEnabled then enableFreecam() else disableFreecam() end
	end
})


-- === ВКЛАДКА HELPER ===
local HelperGroupBox = Tabs.Helper:AddLeftGroupbox("Helper")
local antiDeathEnabled = false
local isEating = false 

task.spawn(function()
	while task.wait(0.1) do
		if Library.Unloaded then break end
		if antiDeathEnabled and not isEating and isCharacterReady() then
			pcall(function()
				local character = player.Character
				local humanoid = character:FindFirstChildOfClass("Humanoid")
				if humanoid and (humanoid.Health <= 15 or humanoid:GetState() == Enum.HumanoidStateType.Ragdoll) then
					local backpack = player:FindFirstChild("Backpack")
					if backpack then
						local foodItem = nil
						for _, item in ipairs(backpack:GetChildren()) do
							if item:IsA("Tool") and (item.Name:lower():find("food") or item.Name:lower():find("hamburger") or item.Name:lower():find("cola") or item.Name:lower():find("eat")) then
								foodItem = item
								break
							end
						end
						if foodItem then
							isEating = true
							humanoid:EquipTool(foodItem)
							task.wait(0.1)
							for i = 1, 3 do
								if foodItem and foodItem.Parent == character then foodItem:Activate() task.wait(0.2) end
							end
							isEating = false
						end
					end
				end
			end)
		end
	end
end)

HelperGroupBox:AddToggle("AntiDeathToggle", {
	Text = "Enable Anti Death", Default = false,
	Callback = function(Value) antiDeathEnabled = Value end
})

-- ANTI-KICK BYPASS V2
task.spawn(function()
	while task.wait(0.3) do
		if Library.Unloaded then break end
		if antiKickV2Enabled then
			pcall(function()
				local mt = getrawmetatable(game)
				if mt and setreadonly then
					setreadonly(mt, false)
					local oldNewIndex = mt.__newindex
					mt.__newindex = function(t, k, v)
						if antiKickV2Enabled and t == player and (k == "Kick" or k == "kick") then
							return nil
						end
						return oldNewIndex(t, k, v)
					end
					rawset(player, "Kick", function() return nil end)
					rawset(player, "kick", function() return nil end)
					setreadonly(mt, true)
				end
			end)
		end
	end
end)

local AntiKickV2Button
AntiKickV2Button = HelperGroupBox:AddButton({
	Text = "anti kick Bypass V2: Off",
	Func = function()
		antiKickV2Enabled = not antiKickV2Enabled
		AntiKickV2Button:SetText(antiKickV2Enabled and "anti kick Bypass V2: On" or "anti kick Bypass V2: Off")
	end
})


-- === ВКЛАДКА COMBAT ===
local CombatGroupBox = Tabs.Combat:AddLeftGroupbox("Combat")
local TriggerBotGroupBox = Tabs.Combat:AddLeftGroupbox("TriggerBot")
local HitboxGroupBox = Tabs.Combat:AddRightGroupbox("hit box")

local autoComboEnabled = false

local function stopAttackAnims(char)
	local humanoid = char:FindFirstChildOfClass("Humanoid")
	if humanoid then
		for _, track in pairs(humanoid:GetPlayingAnimationTracks()) do
			local name = track.Name:lower()
			if name:find("attack") or name:find("shoot") or name:find("swing") or name:find("tool") or name:find("slash") then
				track:Stop(0)
			end
		end
	end
end

-- HITBOX LOGIC
task.spawn(function()
	while task.wait(0.3) do
		if Library.Unloaded then break end
		if hitboxEnabled then
			pcall(function()
				for _, v in ipairs(game:GetService("Players"):GetPlayers()) do
					if v ~= player and v.Character then
						local hrp = v.Character:FindFirstChild("HumanoidRootPart")
						local humanoid = v.Character:FindFirstChildOfClass("Humanoid")
						if hrp and humanoid and humanoid.Health > 0 then
							hrp.Size = Vector3.new(hitboxSize, hitboxSize, hitboxSize)
							hrp.Shape = Enum.PartType.Ball 
							hrp.Color = Color3.new(1, 1, 1) 
							hrp.Transparency = 0.93 
							hrp.CanCollide = false
						end
					end
				end
			end)
		end
	end
end)

-- AUTO COMBO LOGIC
runService.Stepped:Connect(function()
	if autoComboEnabled and isCharacterReady() then
		pcall(function()
			local character = player.Character
			local backpack = player:FindFirstChild("Backpack")
			if character and backpack then
				local pickaxe = character:FindFirstChild("Pickaxe") or backpack:FindFirstChild("Pickaxe")
				local axe = character:FindFirstChild("Axe") or backpack:FindFirstChild("Axe")
				if character:FindFirstChild("Pickaxe") or character:FindFirstChild("Axe") then
					stopAttackAnims(character)
					if pickaxe and pickaxe.Parent ~= character then pickaxe.Parent = character end
					if axe and axe.Parent ~= character then axe.Parent = character end
					if pickaxe then pickaxe:Activate() end
					if axe then axe:Activate() end
				end
			end
		end)
	end
end)

-- FAST ATTACK LOGIC
runService.RenderStepped:Connect(function()
	if not isCharacterReady() then return end
	local char = player.Character
	if autoComboEnabled and (char:FindFirstChild("Pickaxe") or char:FindFirstChild("Axe")) then stopAttackAnims(char) end

	if fastAttackEnabled then
		pcall(function()
			local backpack = player.Backpack
			local holdingTool = char:FindFirstChildOfClass("Tool")
			if holdingTool and (holdingTool.Name:lower():find("musket") or holdingTool.Name:lower():find("stop") or holdingTool.Name:lower():find("mustak")) then
				stopAttackAnims(char)
				for _, tool in pairs(backpack:GetChildren()) do
					if tool.Name:lower():find("musket") or tool.Name:lower():find("stop") or tool.Name:lower():find("mustak") then
						tool.Parent = char
						tool.Grip = CFrame.new(0, 0, 0)
					end
				end
				
				for _, tool in pairs(char:GetChildren()) do
					if tool:IsA("Tool") and (tool.Name:lower():find("musket") or tool.Name:lower():find("stop") or tool.Name:lower():find("mustak")) then
						tool:Activate()
						for _, v in pairs(game.Players:GetPlayers()) do
							if v ~= game.Players.LocalPlayer and v.Character then
								local targetHead = v.Character:FindFirstChild("Head")
								local targetHumanoid = v.Character:FindFirstChild("Humanoid")
								if targetHead and targetHumanoid and targetHumanoid.Health > 0 then
									local dist = (player.Character.HumanoidRootPart.Position - targetHead.Position).Magnitude
									if dist <= auraRadius then
										if not targetHead:FindFirstChild("MagmaInject") then
											local p = Instance.new("Part")
											p.Name = "MagmaInject"
											p.Shape = Enum.PartType.Ball
											p.Size = Vector3.new(hbSize, hbSize, hbSize)
											p.Transparency = 0.95
											p.CanCollide = false
											p.Massless = true
											p.Parent = targetHead

											local w = Instance.new("WeldPartAdornment")
											w.Adornee = targetHead
											w.Part0 = targetHead
											w.Part1 = p
											w.C0 = CFrame.new(0, 0, 0)
											w.Parent = p
										end
										tool:Activate()
										if math.random(1, 100) > 85 then task.wait(0.05) end
									end
								end
							end
						end
					end
				end
			end
		end)
	end
end)

task.spawn(function()
	while task.wait(0.5) do
		if Library.Unloaded then break end
		if fastAttackEnabled and isCharacterReady() then
			pcall(function()
				local repStorage = game:GetService("ReplicatedStorage")
				local modules = repStorage:FindFirstChild("Modules")
				local weaponModFile = modules and modules:FindFirstChild("WeaponModule")
				if weaponModFile then
					local WeaponModule = require(weaponModFile)
					if WeaponModule and WeaponModule.CD then
						for i, v in pairs(WeaponModule.CD) do
							if i:lower():find("musket") or i:lower():find("stop") or i:lower():find("mustak") then WeaponModule.CD[i] = safeSpeed end
						end
					end
				end
				local char = player.Character
				if char then
					for _, tool in pairs(char:GetChildren()) do
						if tool:IsA("Tool") and tool:FindFirstChild("CD") then tool.CD.Value = safeSpeed + math.random(0.01, 0.03) end
					end
				end
			end)
		end
	end
end)

runService.Heartbeat:Connect(function(step)
	if step > 0.04 then safeSpeed = 0.35 else safeSpeed = 0.29 end
end)


-- === TRIGGERBOT ===
task.spawn(function()
	while task.wait(0.1) do
		if Library.Unloaded then break end
		if (triggerBotEnabled or triggerBot2Enabled) and isCharacterReady() then
			pcall(function()
				local myHrp = player.Character.HumanoidRootPart
				local targetFound = false
				
				for _, p in ipairs(game:GetService("Players"):GetPlayers()) do
					if p ~= player and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
						local enemyHrp = p.Character.HumanoidRootPart
						if (myHrp.Position - enemyHrp.Position).Magnitude <= triggerBotRadius then
							local enemyHum = p.Character:FindFirstChildOfClass("Humanoid")
							if enemyHum and enemyHum.Health > 0 then
								targetFound = true
								break
							end
						end
					end
				end
				
				if targetFound then
					local currentChar = player.Character
					
					if triggerBotEnabled and player:FindFirstChild("Backpack") then
						for _, tool in ipairs(player.Backpack:GetChildren()) do
							local n = tool.Name:lower()
							if n:find("musket") or n:find("stop") or n:find("mustak") or n:find("pickaxe") or n:find("axe") then
								currentChar:FindFirstChildOfClass("Humanoid"):EquipTool(tool)
								break
							end
						end
						for _, tool in ipairs(currentChar:GetChildren()) do
							if tool:IsA("Tool") then tool:Activate() end
						end
					end
					
					if triggerBot2Enabled then
						local currentTool = currentChar:FindFirstChildOfClass("Tool")
						if currentTool then currentTool:Activate() end
					end
				end
			end)
		end
	end
end)


-- === ИНТЕРФЕЙС COMBAT ===
CombatGroupBox:AddToggle("AutoComboToggle", {
	Text = "Auto Combo (Pickaxe + Axe)", Default = false,
	Callback = function(Value)
		autoComboEnabled = Value
		if not Value and isCharacterReady() then
			pcall(function()
				local backpack = player:FindFirstChild("Backpack")
				if backpack then
					local p = player.Character:FindFirstChild("Pickaxe")
					local a = player.Character:FindFirstChild("Axe")
					if p then p.Parent = backpack end
					if a then a.Parent = backpack end
				end
			end)
		end
	end
})

CombatGroupBox:AddToggle("FastAttackToggle", {
	Text = "Fast Attack (Musket + Road Sign)", Default = false,
	Callback = function(Value)
		fastAttackEnabled = Value
		if not Value and isCharacterReady() and player:FindFirstChild("Backpack") then
			pcall(function()
				for _, tool in pairs(player.Character:GetChildren()) do
					if tool:IsA("Tool") and (tool.Name:lower():find("musket") or tool.Name:lower():find("stop") or tool.Name:lower():find("mustak")) then tool.Parent = player.Backpack end
				end
			end)
		end
	end
})

TriggerBotGroupBox:AddToggle("TriggerBotToggle", { Text = "Enable TriggerBot", Default = false, Callback = function(Value) triggerBotEnabled = Value end })
TriggerBotGroupBox:AddToggle("TriggerBot2Toggle", { Text = "Enable TriggerBot 2", Default = false, Callback = function(Value) triggerBot2Enabled = Value end })

HitboxGroupBox:AddToggle("HitboxToggle", {
	Text = "Enable Hitbox", Default = false,
	Callback = function(Value)
		hitboxEnabled = Value
		if not Value then
			pcall(function()
				for _, v in ipairs(game:GetService("Players"):GetPlayers()) do
					if v ~= player and v.Character then
						local hrp = v.Character:FindFirstChild("HumanoidRootPart")
						if hrp then hrp.Size = Vector3.new(2, 2, 1) hrp.Shape = Enum.PartType.Block hrp.Transparency = 1 end
					end
				end
			end)
		end
	end
})

HitboxGroupBox:AddSlider("HitboxSlider", {
	Text = "Hitbox Size", Default = 12, Min = 2, Max = 50, Rounding = 0, Compact = false,
	Callback = function(Value) hitboxSize = Value end
})
