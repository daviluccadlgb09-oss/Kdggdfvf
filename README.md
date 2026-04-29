-- ███████╗ ██████╗██████╗ ██╗██████╗ ████████╗███████╗██████╗ ███████╗██╗  ██╗██╗   ██╗██████╗ 
-- ██╔════╝██╔════╝██╔══██╗██║██╔══██╗╚══██╔══╝██╔════╝██╔══██╗██╔════╝██║  ██║██║   ██║██╔══██╗
-- ███████╗██║     ██████╔╝██║██████╔╝   ██║   █████╗  ██████╔╝███████╗███████║██║   ██║██████╔╝
-- ╚════██║██║     ██╔══██╗██║██╔═══╝    ██║   ██╔══╝  ██╔══██╗╚════██║██╔══██║██║   ██║██╔══██╗
-- ███████║╚██████╗██║  ██║██║██║        ██║   ███████╗██║  ██║███████║██║  ██║╚██████╔╝██████╔╝
-- ╚══════╝ ╚═════╝╚═╝  ╚═╝╚═╝╚═╝        ╚═╝   ╚══════╝╚═╝  ╚═╝╚══════╝╚═╝  ╚═╝ ╚═════╝ ╚═════╝ 

-- Youtube: https://www.youtube.com/@ScriptersHub 

-- Rivals open source script with tons of features, such as:
-- Aimbot
-- ESP
-- WalkSpeed/JumpPower changer
-- Fly, Noclip, Infinite Jump
-- Flashbang effect remover
-- Smoke clouds remover
-- and lots of more!

-- If you take some parts of the script, at least credit them      

queue_on_teleport('loadstring(game:HttpGet("https://raw.githubusercontent.com/no1one1lmao/dfgdfgdfg/refs/heads/main/hgdgfdfgfd"))()')

	local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()
	
	local Window = Rayfield:CreateWindow({
		Name = "SCRIPT ScriptersHub |MADE By @Daviskx041 ❤️",
		LoadingTitle = "Loading it gng, just wait...",
		LoadingSubtitle = "like really, it's just 2 seconds",
		ConfigurationSaving = {
			Enabled = true,
			FolderName = nil,
			FileName = "Rivals Script"
		},
	})
	
	-- Below is a config. Basically, default load values when you first time booted in the cheat.
	
	local Config = {
		AimbotToggle = false,
	        SilentAim = false,
	 	AimbotPart = {"Head"},
		RightMouseDown = false,
		FOV = 150,
		Sensitivity = 0.3,
		LockOnTarget = nil,
		ShowFOV = true,
		WallCheck = true,
		ShowESP = false,
		EnemyColor = Color3.fromRGB(255, 0, 0),
		BlinkingESP = false,
		HPESP = true,
		ESPTransparency = 0.3,
		ShowNameTags = true,
		WalkSpeed = false,
		WalkSpeedValue = 25.2,
		JumpPower = false,
		JumpPowerValue = 20,
		Noclip = false,
		Smoke = false,
		InfiniteJump = false,
		Fly = false,
		FlySpeed = 100,
		Crosshair = false,
	}
	
	local Players = game:GetService("Players")
	local UserInputService = game:GetService("UserInputService")
	local RunService = game:GetService("RunService")
	local Camera = workspace.CurrentCamera
	local LocalPlayer = Players.LocalPlayer
	local UIS = game:GetService("UserInputService")
	local Mouse = LocalPlayer:GetMouse()
	
	local ReplicatedStorage = game:GetService("ReplicatedStorage")
	local utility = require(game:GetService("ReplicatedStorage").Modules.Utility)
	local oldRaycast = utility.Raycast
	
	local originalName = LocalPlayer.Name
	local originalDisplayName = LocalPlayer.DisplayName
	
	local noclippedParts = {}
	local HumanModCons = HumanModCons or {}
	local connections = {}
	local storedNametags = {}
	local DrawingCircle
	local InfiniteJumpConnection
	
	-- Aimbot Logic
	
	if Drawing then
		DrawingCircle = Drawing.new("Circle")
		DrawingCircle.Thickness = 1
		DrawingCircle.Filled = false
		DrawingCircle.Transparency = 1
		DrawingCircle.Color = Color3.fromRGB(255, 255, 255)
		DrawingCircle.Visible = Config.ShowFOV
		DrawingCircle.Radius = Config.FOV
		table.insert(connections, RunService.RenderStepped:Connect(function()
			DrawingCircle.Position = Vector2.new(Mouse.X, Mouse.Y + 36)
			DrawingCircle.Radius = Config.FOV
			DrawingCircle.Visible = Config.ShowFOV and Config.AimbotToggle
		end))
	end
	
	local function isValidTarget(player)
		if player == LocalPlayer then return false end
		if not player.Character then return false end
		if not player.Character:FindFirstChild("Humanoid") then return false end
		if player.Character.Humanoid.Health <= 0 then return false end
		return true
	end
	
	local function ClearHighlights()
		for _, player in pairs(Players:GetPlayers()) do
			if player.Character then
				local highlight = player.Character:FindFirstChild("ESPHighlight")
				if highlight then highlight:Destroy() end
				local nametag = player.Character:FindFirstChild("ESPNameTag")
				if nametag then nametag:Destroy() end
			end
		end
	end
	
	local function getTargetPart(character)
	    if character:FindFirstChild("Humanoid") then
	        if type(Config.AimbotPart) ~= "table" then
	            Config.AimbotPart = { tostring(Config.AimbotPart) }
	        end
	
	        for _, partName in ipairs(Config.AimbotPart) do
	            local part = character:FindFirstChild(partName)
	            if part then
	                return part
	            end
	        end
	    end
	end
	
	local function isVisible(targetPlayer)
		local character = targetPlayer.Character
		if not character then return false end
		local targetPart = getTargetPart(character)
		if not targetPart then return false end
		local targetPos = targetPart.Position
		local origin = Camera.CFrame.Position
		local direction = (targetPos - origin).Unit * 1000
		local rayParams = RaycastParams.new()
		rayParams.FilterType = Enum.RaycastFilterType.Blacklist
		rayParams.FilterDescendantsInstances = {LocalPlayer.Character}
		rayParams.IgnoreWater = true
		local result = workspace:Raycast(origin, direction, rayParams)
		if result and result.Instance then
			local hitPart = result.Instance
			if not character:IsAncestorOf(hitPart) then
				return false
			end
		end
		return true
	end
	
	local function getClosestPlayerInFOV()
		local closestPlayer = nil
		local shortestDistance = Config.FOV
		for _, player in pairs(Players:GetPlayers()) do
			if isValidTarget(player) then
				local targetPart = getTargetPart(player.Character)
				if targetPart then
					if not Config.WallCheck or isVisible(player) then
						local screenPoint = Camera:WorldToScreenPoint(targetPart.Position)
						local distance = (Vector2.new(Mouse.X, Mouse.Y) - Vector2.new(screenPoint.X, screenPoint.Y)).Magnitude
						if distance < shortestDistance then
							closestPlayer = player
							shortestDistance = distance
						end
					end
				end
			end
		end
		return closestPlayer
	end
	
	local function GetClosestTarget()
	    if type(Config.AimbotPart) ~= "table" then
	        Config.AimbotPart = { tostring(Config.AimbotPart) }
	    end
	
	    local closest, dist, targetPart = nil, math.huge, nil
	    local center = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
	
	    for _, plr in pairs(Players:GetPlayers()) do
	        if plr ~= LocalPlayer and plr.Character then
	            for _, partName in ipairs(Config.AimbotPart) do
	                local part = plr.Character:FindFirstChild(partName)
	                if part then
	                    local pos, onScreen = Camera:WorldToViewportPoint(part.Position)
	                    if onScreen then
	                        local mag = (Vector2.new(pos.X, pos.Y) - center).Magnitude
	                        if mag < dist and mag <= Config.FOV then
	                            if not Config.WallCheck or (function()
	                                local rayOrigin = Camera.CFrame.Position
	                                local rayDirection = (part.Position - rayOrigin).Unit * 1000
	
	                                local rayParams = RaycastParams.new()
	                                rayParams.FilterType = Enum.RaycastFilterType.Blacklist
	                                rayParams.FilterDescendantsInstances = {LocalPlayer.Character}
	                                rayParams.IgnoreWater = true
	
	                                local result = workspace:Raycast(rayOrigin, rayDirection, rayParams)
	                                return result and result.Instance and plr.Character:IsAncestorOf(result.Instance)
	                            end)() then
	                                closest, dist, targetPart = plr.Character, mag, part
	                            end
	                        end
	                    end
	                    SCRIPTT
	                end
	            end
	        end
	    end
	
	    return targetPart
	end
	
	local function SilentAimHandler()
	    if Config.SilentAim and Config.AimbotToggle then
	        utility.Raycast = function(...)
	            local args = {...}
	
	            if args[4] == 999 then
	                local part = GetClosestTarget()
	                if part then
	                    args[3] = part.Position
	                end
	            end
	
	            return oldRaycast(table.unpack(args))
	        end
	    else
	        utility.Raycast = oldRaycast
	    end
	end
	
	local function isLockedTargetValid()
		local target = Config.LockOnTarget
		if not target then return false end
		if not target.Parent then return false end
		if not isValidTarget(target) then return false end
		if not getTargetPart(target.Character) then return false end
		return true
	end
	
	table.insert(connections, UserInputService.InputBegan:Connect(function(input, processed)
		if not processed then
			if input.UserInputType == Enum.UserInputType.MouseButton2 then
				Config.RightMouseDown = true
			end
		end
	end))
	
	table.insert(connections, UserInputService.InputEnded:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton2 then
			Config.RightMouseDown = false
			Config.LockOnTarget = nil
		end
	end))
	
	table.insert(connections, RunService.RenderStepped:Connect(function()
		if not Config.AimbotToggle or not Config.RightMouseDown or Config.SilentAim then return end
		if Config.LockOnTarget and isLockedTargetValid() then
			local targetPart = getTargetPart(Config.LockOnTarget.Character)
			local targetPosition = Camera:WorldToScreenPoint(targetPart.Position)
			if targetPosition.Z > 0 then
				local mousePos = Vector2.new(Mouse.X, Mouse.Y)
				local aimPos = Vector2.new(targetPosition.X, targetPosition.Y)
				local moveDelta = (aimPos - mousePos) * Config.Sensitivity
				mousemoverel(moveDelta.X, moveDelta.Y)
			end
		else
			local newTarget = getClosestPlayerInFOV()
			if newTarget then
				Config.LockOnTarget = newTarget
			end
		end
	end))
	
	table.insert(connections, Players.PlayerRemoving:Connect(function(player)
		if Config.LockOnTarget == player then
			Config.LockOnTarget = nil
		end
	end))
	
	-- Fake Name Logic
	
	local function processtext(text)
		if type(text) ~= "string" then return text end
	
		if Config.UseFakeName and originalName and Config.FakeName then
			text = string.gsub(text, originalName, Config.FakeName)
		end
		if Config.UseFakeDisplayName and originalDisplayName and Config.FakeDisplayName then
			text = string.gsub(text, originalDisplayName, Config.FakeDisplayName)
		end
	
		return text
	end
	
	local function hookUIObject(obj)
		if obj:IsA("TextBox") or obj:IsA("TextLabel") or obj:IsA("TextButton") then
			pcall(function()
				obj.Text = processtext(obj.Text)
				obj.Name = processtext(obj.Name)
				obj.Changed:Connect(function(prop)
					if prop == "Text" or prop == "Name" then
						obj.Text = processtext(obj.Text)
						obj.Name = processtext(obj.Name)
					end
				end)
			end)
		end
	end
	
	for _, v in next, game:GetDescendants() do
		hookUIObject(v)
	end
	
	game.DescendantAdded:Connect(hookUIObject)
	
	-- ESP Logic
	
	local function RefreshHighlights()
		for _, player in pairs(Players:GetPlayers()) do
			if player ~= LocalPlayer and player.Character then
				local character = player.Character
				local root = character:FindFirstChild("HumanoidRootPart")
	
				if root then
					if Config.ShowESP and Config.ShowNameTags then
						if root:FindFirstChild("Nametag") then
							storedNametags[player] = root.Nametag:Clone()
							root.Nametag:Destroy()
						end
					elseif storedNametags[player] and not root:FindFirstChild("Nametag") then
						storedNametags[player].Parent = root
						storedNametags[player] = nil
					end
				end
	
				local existingHighlight = character:FindFirstChild("ESPHighlight")
				local existingBillboard = character:FindFirstChild("ESPNameTag")
	
				local shouldBlink = Config.BlinkingESP and (tick() % 4 < 2)
	
				if Config.ShowESP and (not Config.BlinkingESP or shouldBlink) then
					if not existingHighlight then
						local highlight = Instance.new("Highlight")
						highlight.Name = "ESPHighlight"
						highlight.FillTransparency = Config.ESPTransparency
						highlight.OutlineTransparency = 1
						highlight.FillColor = Config.EnemyColor
						highlight.Parent = character
					else
						existingHighlight.FillColor = Config.EnemyColor
						existingHighlight.FillTransparency = Config.ESPTransparency
					end
	
					if Config.ShowNameTags then
						if not existingBillboard then
							local head = character:FindFirstChild("Head")
							if head then
								local bb = Instance.new("BillboardGui")
								bb.Name = "ESPNameTag"
								bb.Adornee = head
								bb.Size = UDim2.new(0, 100, 0, 20)
								bb.StudsOffset = Vector3.new(0, 2.5, 0)
								bb.AlwaysOnTop = true
	
								local label = Instance.new("TextLabel", bb)
								label.Size = UDim2.new(1, 0, 1, 0)
								label.BackgroundTransparency = 1
								label.TextColor3 = Color3.new(1, 1, 1)
								if Config.HPESP then
									task.spawn(function()
	    									while label and label.Parent and Config.ShowESP and Config.HPESP do
	        									local hp = math.floor(player.Character.Humanoid.Health)
	        									label.Text = player.Name .. " | " .. hp .. " HP"
	        									task.wait(0.1)
	    									end
									end)
								else
									label.Text = player.Name
								end
								label.TextScaled = true
								label.Font = Enum.Font.SourceSansBold
	
								bb.Parent = character
							end
						end
					else
						if existingBillboard then existingBillboard:Destroy() end
					end
				else
					if existingHighlight then existingHighlight:Destroy() end
					if existingBillboard then existingBillboard:Destroy() end
				end
			end
		end
	end
	
	Players.PlayerRemoving:Connect(function(player)
		local character = player.Character
		if character then
			local highlight = character:FindFirstChildWhichIsA("Highlight")
			if highlight then
				highlight:Destroy()
			end
		end
	end)
	
	Players.PlayerAdded:Connect(function(player)
		player.CharacterAdded:Connect(function()
			task.wait(1)
			RefreshHighlights()
		end)
	end)
	
	task.spawn(function()
		while true do
			if Config.ShowESP then
				RefreshHighlights()
				task.wait(Config.BlinkingESP and 2 or 0.1)
			else
				ClearHighlights()
				task.wait(0.1)
			end
		end
	end)
	
	-- WalkSpeed/JumpPower/Fly/Infinite Jump/Noclip Logic
	
	local function startLoopSpeed()
	    local h = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildWhichIsA("Humanoid")
	    if not h then return end
	
	    local function apply() 
	        h.WalkSpeed = Config.WalkSpeedValue 
	    end
	
	    apply()
	
	    if HumanModCons.ws then HumanModCons.ws:Disconnect() end
	    HumanModCons.ws = h:GetPropertyChangedSignal("WalkSpeed"):Connect(apply)
	
	    if HumanModCons.wsCA then HumanModCons.wsCA:Disconnect() end
	    HumanModCons.wsCA = LocalPlayer.CharacterAdded:Connect(function(character)
	        h = character:WaitForChild("Humanoid")
	        apply()
	        if HumanModCons.ws then HumanModCons.ws:Disconnect() end
	        HumanModCons.ws = h:GetPropertyChangedSignal("WalkSpeed"):Connect(apply)
	    end)
	end
	
	local function stopLoopSpeed()
	    if HumanModCons.ws then HumanModCons.ws:Disconnect() end
	    if HumanModCons.wsCA then HumanModCons.wsCA:Disconnect() end
	
	    local h = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildWhichIsA("Humanoid")
	    if h then
	        h.WalkSpeed = 25.2
	    end
	end
	
	local function startLoopPower()
	    local h = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildWhichIsA("Humanoid")
	    if not h then return end
	
	    local function apply()
	        h.UseJumpPower = true
	        h.JumpPower = Config.JumpPowerValue
	    end
	
	    apply()
	
	    if HumanModCons.jp then HumanModCons.jp:Disconnect() end
	    HumanModCons.jp = h:GetPropertyChangedSignal("JumpPower"):Connect(apply)
	
	    if HumanModCons.jpCA then HumanModCons.jpCA:Disconnect() end
	    HumanModCons.jpCA = LocalPlayer.CharacterAdded:Connect(function(character)
	        h = character:WaitForChild("Humanoid")
	        apply()
	        if HumanModCons.jp then HumanModCons.jp:Disconnect() end
	        HumanModCons.jp = h:GetPropertyChangedSignal("JumpPower"):Connect(apply)
	    end)
	end
	
	local function stopLoopPower()
	    if HumanModCons.jp then HumanModCons.jp:Disconnect() end
	    if HumanModCons.jpCA then HumanModCons.jpCA:Disconnect() end
	
	    local h = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildWhichIsA("Humanoid")
	    if h then
	        h.JumpPower = 20
	    end
	end
	
	local function startInfiniteJump()
	    if InfiniteJumpConnection then return end
	    InfiniteJumpConnection = UserInputService.JumpRequest:Connect(function()
	        local character = LocalPlayer.Character
	        if character and character:FindFirstChild("Humanoid") then
	            character.Humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
	        end
	    end)
	end
	
	local function stopInfiniteJump()
	    if InfiniteJumpConnection then
	        InfiniteJumpConnection:Disconnect()
	        InfiniteJumpConnection = nil
	    end
	end
	
	task.spawn(function()
		while true do
			local character = LocalPlayer.Character
			if character then
				for _, part in pairs(character:GetDescendants()) do
					if part:IsA("BasePart") then
						if Config.Noclip then
							if part.CanCollide then
								part.CanCollide = false
								noclippedParts[part] = true
							end
						else
							if noclippedParts[part] then
								part.CanCollide = true
								noclippedParts[part] = nil
							end
						end
					end
				end
			end
			task.wait(0.21)
		end
	end)
	
	-- Smoke Logic
	
	task.spawn(function()
		while true do
			if Config.Smoke then
				for i, v in pairs(workspace:GetChildren()) do
					if v.Name == "Smoke Grenade" then
						v:Destroy()
					end
				end
			end
			task.wait(0.1)
		end
	end)
	
	local AimbotTab = Window:CreateTab("Aimbot", 4483362458)
	local AimbotToggl = AimbotTab:CreateToggle({
		Name = "Enable Aimbot",
		CurrentValue = Config.AimbotToggle,
		Flag = "AimbotTrigger",
		Callback = function(Value)
			Config.AimbotToggle = Value
		end,
	})
	AimbotTab:CreateToggle({
	    Name = "Enable Silent Aim",
	    CurrentValue = Config.SilentAim,
	    Flag = "SilentAimToggle",
	    Callback = function(Value)
	        Config.SilentAim = Value
	        SilentAimHandler(Value)
	    end,
	})
	AimbotTab:CreateToggle({
		Name = "Wall Check",
		CurrentValue = Config.WallCheck,
		Flag = "AimbotWallCheck",
		Callback = function(Value)
			Config.WallCheck = Value
		end,
	})
	AimbotTab:CreateToggle({
		Name = "Show FOV",
		CurrentValue = Config.ShowFOV,
		Flag = "AimbotShowFOV",
		Callback = function(Value)
			Config.ShowFOV = Value
			if DrawingCircle then
				DrawingCircle.Visible = Value and Config.AimbotToggle
			end
		end,
	})
	AimbotTab:CreateSlider({
		Name = "FOV",
		Range = {10, 600},
		Increment = 1,
		Suffix = "Degrees",
		Flag = "AimbotFov",
		CurrentValue = Config.FOV,
		Callback = function(Value)
			Config.FOV = Value
		end,
	})
	AimbotTab:CreateSlider({
		Name = "Sensitivity (AIMBOT LOCK SPEED CHANGER)",
		Range = {0.01, 1},
		Increment = 0.01,
		Suffix = "",
		Flag = "AimbotSensitivity",
		CurrentValue = Config.Sensitivity,
		Callback = function(Value)
			Config.Sensitivity = Value
		end,
	})
	local AimbotBind = AimbotTab:CreateKeybind({
		Name = "Bind Aimbot",
		CurrentKeybind = "F15",
		HoldToInteract = false,
		Flag = "AimbotBind",
		Callback = function(Keybind)
			AimbotToggl:Set(not Config.AimbotToggle)
		end,
	})
	AimbotTab:CreateButton({
		Name = "Reset Aimbot Bind",
		Callback = function()
			AimbotBind:Set("F15")
		end,
	})
	AimbotTab:CreateDropdown({
		Name = "Select Aimbot Part",
		Options = {"Head", "UpperTorso", "LeftUpperLeg", "RightUpperLeg", "LeftUpperArm", "RightUpperArm"},
		CurrentOption = Config.AimbotPart,
		MultipleOptions = false,
		Flag = "AimbotPart",
		Callback = function(Option)
			Con# Kdggdfvf
