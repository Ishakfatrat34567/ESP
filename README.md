# ESP
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")

-- STATE
local espEnabled = false
local teamCheck = false
local fillTransparency = 1
local outlineTransparency = 0
local outlineColor = Color3.fromRGB(255, 0, 0)
local boxColor = Color3.fromRGB(255, 0, 0)
local boxTransparency = 0.5
local nameTagColor = Color3.new(1, 1, 1)

-- GUI
local screenGui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
screenGui.Name = "IshkebESP_UI"
screenGui.IgnoreGuiInset = true
screenGui.ResetOnSpawn = false
screenGui.Enabled = true

local panel = Instance.new("Frame", screenGui)
panel.Size = UDim2.new(0, 300, 0, 380)
panel.Position = UDim2.new(0, 20, 0, 100)
panel.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
panel.BorderSizePixel = 0
panel.Active = true
panel.Draggable = true
panel.ZIndex = 10

-- Title
local title = Instance.new("TextLabel", panel)
title.Size = UDim2.new(1, 0, 0, 40)
title.Position = UDim2.new(0, 0, 0, 0)
title.BackgroundTransparency = 1
title.Text = "ishkeb's ESP"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.Font = Enum.Font.SourceSansBold
title.TextScaled = true
title.ZIndex = 12

-- Tabs
local tabHolder = Instance.new("Frame", panel)
tabHolder.Size = UDim2.new(1, 0, 0, 40)
tabHolder.Position = UDim2.new(0, 0, 0, 40) -- moved down
tabHolder.BackgroundTransparency = 1
tabHolder.ZIndex = 11

local function createTabButton(text)
	local btn = Instance.new("TextButton", tabHolder)
	btn.Size = UDim2.new(0.5, 0, 1, 0)
	btn.Text = text
	btn.Font = Enum.Font.SourceSansBold
	btn.TextScaled = true
	btn.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
	btn.TextColor3 = Color3.new(1,1,1)
	btn.ZIndex = 11
	return btn
end

local mainTabButton = createTabButton("Main")
mainTabButton.Position = UDim2.new(0, 0, 0, 0)
local customTabButton = createTabButton("Customize")
customTabButton.Position = UDim2.new(0.5, 0, 0, 0)

local function createTabFrame(parent)
	local tab = Instance.new("Frame", parent)
	tab.Size = UDim2.new(1, 0, 1, -80)
	tab.Position = UDim2.new(0, 0, 0, 80)
	tab.BackgroundTransparency = 1
	tab.ZIndex = 11
	local layout = Instance.new("UIListLayout", tab)
	layout.Padding = UDim.new(0, 10)
	layout.HorizontalAlignment = Enum.HorizontalAlignment.Center
	layout.VerticalAlignment = Enum.VerticalAlignment.Top
	return tab
end

local mainTab = createTabFrame(panel)
local customTab = createTabFrame(panel)
customTab.Visible = false

local function createButton(text, callback, parent)
	local btn = Instance.new("TextButton", parent)
	btn.Size = UDim2.new(0.9, 0, 0, 50)
	btn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
	btn.TextColor3 = Color3.new(1,1,1)
	btn.Text = text
	btn.TextScaled = true
	btn.Font = Enum.Font.SourceSansBold
	btn.ZIndex = 11
	btn.MouseButton1Click:Connect(function() callback(btn) end)
	return btn
end

local function createLabel(text, parent)
	local label = Instance.new("TextLabel", parent)
	label.Size = UDim2.new(0.9, 0, 0, 30)
	label.BackgroundTransparency = 1
	label.TextColor3 = Color3.new(1,1,1)
	label.Font = Enum.Font.SourceSansBold
	label.TextScaled = true
	label.Text = text
	label.ZIndex = 11
	return label
end

-- Refresh all ESP visuals
function refreshESP()
	for _, player in ipairs(Players:GetPlayers()) do
		if player ~= LocalPlayer then
			removeESP(player)
			addESP(player)
		end
	end
end

-- MAIN TAB BUTTONS
createButton("ESP: OFF", function(btn)
	espEnabled = not espEnabled
	btn.Text = espEnabled and "ESP: ON" or "ESP: OFF"
end, mainTab)

createButton("Team Check: OFF", function(btn)
	teamCheck = not teamCheck
	btn.Text = teamCheck and "Team Check: ON" or "Team Check: OFF"
end, mainTab)

-- CUSTOM TAB
createLabel("Outline Transparency:", customTab)
createButton("0.0", function(btn)
	outlineTransparency = math.clamp(outlineTransparency + 0.1, 0, 1)
	btn.Text = string.format("%.1f", outlineTransparency)
	refreshESP()
end, customTab)

createLabel("Quick Colors:", customTab)
createButton("Red", function()
	boxColor = Color3.fromRGB(255, 0, 0)
	refreshESP()
end, customTab)

createButton("Green", function()
	boxColor = Color3.fromRGB(0, 255, 0)
	refreshESP()
end, customTab)

createButton("Blue", function()
	boxColor = Color3.fromRGB(0, 0, 255)
	refreshESP()
end, customTab)

-- Tab Switching
mainTabButton.MouseButton1Click:Connect(function()
	mainTab.Visible = true
	customTab.Visible = false
end)

customTabButton.MouseButton1Click:Connect(function()
	mainTab.Visible = false
	customTab.Visible = true
end)

-- GUI Toggle
local function fadeGui(show)
	local goal = {BackgroundTransparency = show and 0 or 1}
	TweenService:Create(panel, TweenInfo.new(0.3), goal):Play()
end

UserInputService.InputBegan:Connect(function(input, gpe)
	if not gpe and input.KeyCode == Enum.KeyCode.RightControl then
		screenGui.Enabled = not screenGui.Enabled
		fadeGui(screenGui.Enabled)
	end
end)

-- ESP SYSTEM
local function removeESP(player)
	if player.Character then
		local head = player.Character:FindFirstChild("Head")
		if head then
			local tag = head:FindFirstChild("NameTag")
			if tag then tag:Destroy() end
		end

		local highlight = player.Character:FindFirstChild("ESPHighlight")
		if highlight then
			highlight:Destroy()
		end
	end
end

local function addESP(player)
	if player == LocalPlayer then return end
	if teamCheck and player.Team ~= nil and LocalPlayer.Team ~= nil and player.Team == LocalPlayer.Team then return end
	if not player.Character then return end

	local head = player.Character:FindFirstChild("Head")
	if not head then return end

	-- NameTag
	local existingTag = head:FindFirstChild("NameTag")
	if existingTag then
		local label = existingTag:FindFirstChildOfClass("TextLabel")
		if label then
			label.TextColor3 = nameTagColor
		end
	else
		local billboard = Instance.new("BillboardGui", head)
		billboard.Name = "NameTag"
		billboard.Size = UDim2.new(0, 100, 0, 20)
		billboard.StudsOffset = Vector3.new(0, 2.5, 0)
		billboard.AlwaysOnTop = true

		local label = Instance.new("TextLabel", billboard)
		label.Size = UDim2.new(1, 0, 1, 0)
		label.BackgroundTransparency = 1
		label.TextColor3 = nameTagColor
		label.Text = player.Name
		label.TextScaled = true
		label.Font = Enum.Font.SourceSansBold
	end

	-- Highlight Outline
	local existingHighlight = player.Character:FindFirstChild("ESPHighlight")
	if not existingHighlight then
		local highlight = Instance.new("Highlight", player.Character)
		highlight.Name = "ESPHighlight"
		highlight.FillTransparency = 1
		highlight.OutlineTransparency = outlineTransparency
		highlight.OutlineColor = boxColor
	else
		existingHighlight.OutlineTransparency = outlineTransparency
		existingHighlight.OutlineColor = boxColor
	end
end

-- Render loop
RunService.RenderStepped:Connect(function()
	if not espEnabled then
		for _, player in ipairs(Players:GetPlayers()) do
			if player ~= LocalPlayer then removeESP(player) end
		end
		return
	end

	for _, player in ipairs(Players:GetPlayers()) do
		if player ~= LocalPlayer then addESP(player) end
	end
end)

