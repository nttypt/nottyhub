-- Made by: NottyHub

-- Load UI Library
local library = loadstring(game:HttpGet("https://raw.githubusercontent.com/zxciaz/VenyxUI/main/Reuploaded"))()
local venyx = library.new("NottyHub", 5013109572)

-- Themes
local themes = {
    Background = Color3.fromRGB(24, 24, 24),
    Glow = Color3.fromRGB(0, 0, 0),
    Accent = Color3.fromRGB(10, 10, 10),
    LightContrast = Color3.fromRGB(20, 20, 20),
    DarkContrast = Color3.fromRGB(14, 14, 14),  
    TextColor = Color3.fromRGB(255, 255, 255)
}

-- Aimbot Variables
local aimbotEnabled = false
local aimbotKey = Enum.KeyCode.Q
local aimPart = "Head"  -- Default Aim Part
local ignoreFriends = false  -- ✅ Ignore friends toggle
local checkVisibility = false -- ✅ Visibility check toggle
local aiming = false
local fovRadius = 150
local fovVisible = false

-- Services
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- Create FOV Circle
local FOVCircle = Drawing.new("Circle")
FOVCircle.Radius = fovRadius
FOVCircle.Thickness = 2
FOVCircle.Color = Color3.fromRGB(255, 255, 255)
FOVCircle.Transparency = 1
FOVCircle.Filled = false
FOVCircle.Visible = fovVisible

-- Keep FOV Circle Centered on Mouse
RunService.RenderStepped:Connect(function()
    if fovVisible then
        local mousePos = UserInputService:GetMouseLocation()
        FOVCircle.Position = Vector2.new(mousePos.X, mousePos.Y)
        FOVCircle.Radius = fovRadius
    end
end)

-- ✅ Function to Check If Player is Visible
local function isPlayerVisible(targetPart)
    if not checkVisibility then return true end -- If visibility check is disabled, always return true

    local origin = Camera.CFrame.Position
    local direction = (targetPart.Position - origin).Unit * (targetPart.Position - origin).Magnitude

    local raycastParams = RaycastParams.new()
    raycastParams.FilterType = Enum.RaycastFilterType.Blacklist
    raycastParams.FilterDescendantsInstances = {LocalPlayer.Character, Camera} -- Ignore self & camera

    local result = workspace:Raycast(origin, direction, raycastParams)

    return result == nil or result.Instance:IsDescendantOf(targetPart.Parent) -- If nothing is hit OR it hits the target, it's visible
end

-- Aimbot Function
local function getClosestPlayer()
    local closestPlayer = nil
    local shortestDistance = math.huge
    local mousePos = UserInputService:GetMouseLocation()

    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            -- ✅ Ignore friends if the toggle is enabled
            if ignoreFriends and LocalPlayer:IsFriendsWith(player.UserId) then
                continue
            end

            local targetPart = player.Character:FindFirstChild(aimPart)
            if targetPart and isPlayerVisible(targetPart) then -- ✅ Check if the player is visible
                local targetPos = targetPart.Position
                local screenPos, onScreen = Camera:WorldToScreenPoint(targetPos)

                if onScreen then
                    local distance = (Vector2.new(mousePos.X, mousePos.Y) - Vector2.new(screenPos.X, screenPos.Y)).Magnitude
                    if distance < shortestDistance and distance <= fovRadius then
                        shortestDistance = distance
                        closestPlayer = targetPart
                    end
                end
            end
        end
    end
    return closestPlayer
end

local function aimAtTarget()
    while aiming do
        local target = getClosestPlayer()
        if target then
            Camera.CFrame = Camera.CFrame:Lerp(CFrame.new(Camera.CFrame.Position, target.Position), 0.3) -- ✅ Smooth transition
        end
        task.wait()
    end
end

-- Aimbot UI
local page = venyx:addPage("Aiming", 5012544693)
local section1 = page:addSection("Aimbot")
local section2 = page:addSection("Aimbot FOV")

-- Toggle Aimbot
section1:addToggle("Enabled", false, function(value)
    aimbotEnabled = value
    print("Aimbot Enabled:", value)
end)

-- Aimbot Keybind
section1:addKeybind("Keybind", aimbotKey, function()
    if aimbotEnabled then
        aiming = true
        aimAtTarget()
        print("Aimbot Activated")
    end
end, function(key)
    aimbotKey = key.KeyCode
    print("Changed Keybind to:", key.KeyCode.Name)
end)

-- Aim Part Dropdown
section1:addDropdown("Aim Part", {"Head", "UpperTorso", "LowerTorso", "LeftUpperArm", "RightUpperArm", "LeftUpperLeg", "RightUpperLeg"}, function(selected)
    aimPart = selected
    print("Aiming at:", selected)
end)

-- ✅ Friend Check Toggle
section1:addToggle("Friend Check", false, function(value)
    ignoreFriends = value
    print("Ignore Friends:", value)
end)

-- ✅ Visibility Check Toggle
section1:addToggle("Visible Check", false, function(value)
    checkVisibility = value
    print("Check Visibility:", value)
end)

-- Toggle FOV Circle
section2:addToggle("Show FOV Circle", false, function(value)
    fovVisible = value
    FOVCircle.Visible = value
    print("FOV Circle Visible:", value)
end)

-- FOV Radius Slider
section2:addSlider("FOV Radius", 150, 10, 1000, function(value)
    fovRadius = value
    print("FOV Radius Set To:", value)
end)

-- Handle Keybind Activation
UserInputService.InputBegan:Connect(function(input)
    if input.KeyCode == aimbotKey and aimbotEnabled then
        aiming = true
        aimAtTarget()
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.KeyCode == aimbotKey then
        aiming = false
        print("Aimbot Deactivated")
    end
end)

-- Apply UI Theme
library:setTheme("Glow", Color3.fromRGB(168, 106, 255))
library:setTheme("TextColor", Color3.fromRGB(168, 106, 255))

-- Load Default Page
venyx:SelectPage(venyx.pages[1], true)
