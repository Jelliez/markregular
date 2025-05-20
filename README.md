local TweenService = game:GetService("TweenService")
local Players = game:GetService("Players")
local player = Players.LocalPlayer
local Hotbar = player.PlayerGui.Hotbar.Backpack.Hotbar
local UltBar = player.PlayerGui.ScreenGui.MagicHealth.Health.Bar.Bar
local UltGlow = player.PlayerGui.ScreenGui.MagicHealth.Health.Glow
local UltG = player.PlayerGui.ScreenGui.MagicHealth.Ult
local UltText = player.PlayerGui.ScreenGui.MagicHealth.TextLabel
UltText.Text = "YOU DONT LIVE TO SEE TOMORROW"
UltText.TextColor3 = Color3.fromRGB(255, 226, 24) -- Text Color
UltText.Position = UDim2.new(UltText.Position.X.Scale, UltText.Position.X.Offset, UltText.Position.Y.Scale, 22)
UltText.TextSize = 20
UltText.TextLabel.TextSize = 20
UltG.TextSize = 0
UltG.TextLabel.TextSize = 0
local darkBlue = Color3.fromRGB(0, 102, 204) -- Text Shading
local lightBlue = Color3.fromRGB(32, 118, 255) -- Background Color
UltBar.ImageColor3 = lightBlue
UltGlow.ImageColor3 = Color3.fromRGB(111, 111, 255) -- Glow Color

local tweenInfo = TweenInfo.new(
    2.5, -- duration
    Enum.EasingStyle.Sine,
    Enum.EasingDirection.InOut,
    -1, -- loop forever
    true -- reverse
)

local pulseTween = TweenService:Create(UltBar, tweenInfo, {
    ImageColor3 = lightBlue
})

pulseTween:Play()

-- Function to change the skybox and lighting
local function changeSkyAndLighting()
    local Lighting = game:GetService("Lighting")

    -- Remove any existing sky
    for _, child in pairs(Lighting:GetChildren()) do
        if child:IsA("Sky") then
            child:Destroy()
        end
    end

    -- Create a new Sky instance (Classical Sunset Sky)
    local newSky = Instance.new("Sky")
    newSky.Name = "ClassicalSunsetSky"

    -- Apply the same ID to all six sides
    local skyId = "rbxassetid://15502592084"
    newSky.SkyboxBk = skyId
    newSky.SkyboxDn = skyId
    newSky.SkyboxFt = skyId
    newSky.SkyboxLf = skyId
    newSky.SkyboxRt = skyId
    newSky.SkyboxUp = skyId

    -- Parent the new sky to Lighting
    newSky.Parent = Lighting

    -- Set lighting properties for a less orange and more natural effect
    Lighting.TimeOfDay = "16:00:00" -- Near sunset time
    Lighting.OutdoorAmbient = Color3.fromRGB(120, 100, 80) -- Warmer light but not too intense
    Lighting.Ambient = Color3.fromRGB(90, 70, 60) -- A bit darker and less saturated
    Lighting.Brightness = 1.6 -- Slightly lower brightness for a softer effect
    Lighting.EnvironmentDiffuseScale = 0.6
    Lighting.EnvironmentSpecularScale = 0.25

    -- Adjust atmosphere to make the fog less purple and more neutral
    local atmosphere = Instance.new("Atmosphere")
    atmosphere.Density = 0.25 -- Reduced density for less intense fog
    atmosphere.Offset = 0.1 -- Slightly reduced offset for a more natural appearance
    atmosphere.Color = Color3.fromRGB(255, 224, 180) -- A more neutral warm color
    atmosphere.Decay = Color3.fromRGB(180, 140, 100) -- Warmer but not purple
    atmosphere.Parent = Lighting
end

-- Call the function to change the sky and lighting immediately
changeSkyAndLighting()

-- Listen for when the player character is added, and apply the environment settings again
game.Players.LocalPlayer.CharacterAdded:Connect(function()
    task.wait(2)  -- Ensure everything is loaded before running the environment changes
    changeSkyAndLighting()
end)

local player = game.Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()

-- Variable to track if accessories have been added
local accessoriesAdded = false

-- Function to remove all accessories
local function removeAllAccessories()
    -- Only remove accessories the first time
    if accessoriesAdded then
        return
    end

    -- Iterate through all accessories and destroy them
    for _, v in pairs(character:GetChildren()) do
        if v:IsA("Accessory") then
            v:Destroy() -- Remove the accessory
        end
    end

    -- Check for accessories attached to specific parts (Head, Neck, etc.)
    for _, v in pairs(character:GetDescendants()) do
        if v:IsA("Accessory") then
            v:Destroy() -- Destroy accessories attached through welds or attachments
        end
    end
end

-- Function to apply a new accessory
local function applyAccessory(assetId)
    -- First, remove all existing accessories
    removeAllAccessories()

    -- Load the new accessory
    local accessory = game:GetObjects("rbxassetid://" .. assetId)[1]
    if accessory and accessory:IsA("Accessory") then
        accessory.Parent = character -- Parent the accessory to the character

        -- Manually attach the accessory if necessary
        local handle = accessory:FindFirstChild("Handle")
        if handle then
            local attachment = handle:FindFirstChildOfClass("Attachment")
            if attachment then
                local characterAttachment = character:FindFirstChild(attachment.Name, true)
                if characterAttachment then
                    local weld = Instance.new("Weld")
                    weld.Part0 = handle
                    weld.Part1 = characterAttachment.Parent
                    weld.C0 = attachment.CFrame
                    weld.C1 = characterAttachment.CFrame
                    weld.Parent = handle
                end
            end
        end
    else
        warn("Failed to load accessory:", assetId)
    end

    -- Mark that accessories have been added
    accessoriesAdded = true
end

-- Example usage: Apply the new accessories
applyAccessory(114561172605430) -- Face accessory
applyAccessory(82067401783914) -- ears

-- Function to apply clothing
local function applyClothing(assetId, classType)
    -- Remove any existing clothing of the specified class type (Shirt/Pants)
    for _, v in pairs(character:GetChildren()) do
        if v:IsA(classType) then
            v:Destroy() -- Remove the old clothing
        end
    end

    -- Load and apply new clothing
    local asset = game:GetObjects("rbxassetid://" .. assetId)[1]
    if asset and asset:IsA(classType) then
        asset.Parent = character
    end
end

applyClothing(103331948794349, "Shirt") -- Shirt
applyClothing(15815815771, "Pants") -- Pants

-- Function to set skin tone
local function setSkinTone(color)
    if not character then return end

    for _, part in pairs(character:GetChildren()) do
        if part:IsA("BasePart") and part.Name ~= "Handle" then
            part.BrickColor = BrickColor.new(color) -- Set skin tone color
        end
    end
end

-- Example usage: Change skin tone to Light orange
setSkinTone("Light orange")

local player = game.Players.LocalPlayer

local function makeAccessoriesNonCollidable(character)
    for _, accessory in pairs(character:GetChildren()) do
        if accessory:IsA("Accessory") then
            local handle = accessory:FindFirstChild("Handle")
            if handle and handle:IsA("BasePart") then
                handle.CanCollide = false
                handle.Massless = true
            end
        end
    end
end

-- Run on current character
if player.Character then
    makeAccessoriesNonCollidable(player.Character)
end

-- Run again when character respawns
player.CharacterAdded:Connect(function(char)
    char:WaitForChild("Humanoid")
    makeAccessoriesNonCollidable(char)
end)

local player = game.Players.LocalPlayer
local playerGui = player.PlayerGui
local UserInputService = game:GetService("UserInputService")
local mouse = player:GetMouse()

local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local flying = false
local currentAnimTrack = nil  -- Variable to hold the animation track

-- Particle effect variables
local Test = game.ReplicatedStorage.Resources.Omni.Windd.Wind
local test = nil
local particleEmitterActive = false

-- Function to start the particle effect
local function startParticles()
    test = Test:Clone()
    test.Parent = player.Character:WaitForChild("HumanoidRootPart")

    for _, child in ipairs(test:GetChildren()) do
        if child:IsA("ParticleEmitter") then
            child:Emit(15)
            child.Enabled = true
        end
    end
    particleEmitterActive = true
end

-- Function to stop the particle effect
local function stopParticles()
    if test then
        test:Destroy()
        test = nil
    end
    particleEmitterActive = false
end

-- Function to play the animation when the player follows the cursor
local function playAnimation()
    -- Play the animation with ID 16597322398 only when following the cursor
    local anim = Instance.new("Animation")
    anim.AnimationId = "rbxassetid://17889080495"
    local humanoid = player.Character and player.Character:FindFirstChild("Humanoid")
    if humanoid then
        local animTrack = humanoid:LoadAnimation(anim)
        animTrack:Play()
        animTrack:AdjustSpeed(0.00001)  -- Adjust the speed to 0.01 to make it slower
        currentAnimTrack = animTrack  -- Store the animation track for later stopping

    end
end

-- Function to stop the animation if it's playing
local function stopAnimation()
    if currentAnimTrack then
        currentAnimTrack:Stop()
        currentAnimTrack = nil  -- Clear the stored animation track
    end
end

-- Function to make the player follow the cursor with smoother movement and rotation
local function followCursor()
    if flying then return end  -- Prevent multiple activations

    flying = true

    -- Create a BodyVelocity to move the player
    local bodyVelocity = Instance.new("BodyVelocity")
    bodyVelocity.MaxForce = Vector3.new(10000000000000000000, 10000000000000000000, 10000000000000000000)  -- Very high force to ensure movement
    bodyVelocity.Velocity = Vector3.new(0, 0, 0)  -- Start with no velocity
    bodyVelocity.Parent = character:WaitForChild("HumanoidRootPart")

    -- Play animation once the player starts moving
    playAnimation()

    -- Function to update the player's position to follow the mouse
    local function updateMovement()
        local smoothingFactor = 0.2  -- Adjust the smoothing factor for rotation and movement
        while flying do
            local mousePosition = mouse.Hit.p  -- Get the 3D position of the mouse
            local humanoidRootPart = character.HumanoidRootPart

            -- Smooth movement towards the mouse position
            local direction = (mousePosition - humanoidRootPart.Position).unit
            bodyVelocity.Velocity = direction * 75  -- Adjust speed by changing the multiplier

            -- Smoothly rotate the character towards the mouse position (left-right and up-down)
            local lookAtCFrame = CFrame.lookAt(humanoidRootPart.Position, mousePosition)

            -- Manually smooth the rotation for X (horizontal) and Y (vertical) axes
            local newCFrame = humanoidRootPart.CFrame:Lerp(lookAtCFrame, smoothingFactor)

            -- Apply the smooth CFrame (horizontal and vertical look)
            character:SetPrimaryPartCFrame(newCFrame)

            wait(0.01)  -- Update every 0.01 second for smoother movement
        end
    end

    -- Start the movement in a separate thread
    coroutine.wrap(updateMovement)()
end

-- Function to stop flying and stop animation
local function stopFlying()
    flying = false
    -- Stop the flying movement
    local bodyVelocity = character:FindFirstChild("HumanoidRootPart"):FindFirstChildOfClass("BodyVelocity")
    if bodyVelocity then
        bodyVelocity:Destroy()
    end
    stopAnimation()  -- Stop the animation when stopping flying
end

-- Function to detect when the R key is pressed and toggle flying and particle effect
local function onRKeyPressed(input, gameProcessedEvent)
    if gameProcessedEvent then return end  -- Ignore if the event is processed by the game (e.g., typing)

    if input.UserInputType == Enum.UserInputType.Keyboard and input.KeyCode == Enum.KeyCode.R then
        if flying then
            stopFlying()  -- Stop flying and animation if already flying
            stopParticles()  -- Stop particles
        else
            followCursor()  -- Start flying and animation
            startParticles()  -- Start particles
        end
    end
end

-- Connect the input event to trigger the function
UserInputService.InputBegan:Connect(onRKeyPressed)

-- Function to update tool name in hotbar
local function updateToolName(buttonIndex, toolName)
    local hotbar = playerGui:FindFirstChild("Hotbar")
    if hotbar then
        local backpack = hotbar:FindFirstChild("Backpack")
        if backpack then
            local hotbarFrame = backpack:FindFirstChild("Hotbar")
            if hotbarFrame then
                local baseButton = hotbarFrame:FindFirstChild(tostring(buttonIndex)).Base
                if baseButton and baseButton.ToolName then
                    baseButton.ToolName.Text = toolName
                end
            end
        end
    end
end

-- Call to update tool names
updateToolName(1, "Seismic Punch")
updateToolName(2, "Brutal Barage")
updateToolName(3, "Ground Surge")
updateToolName(4, "Chin-checker")

-- Function to set magic health text
local function findGuiAndSetText()
    local screenGui = playerGui:FindFirstChild("ScreenGui")
    if screenGui then
        local magicHealthFrame = screenGui:FindFirstChild("MagicHealth")
        if magicHealthFrame then
            local textLabel = magicHealthFrame:FindFirstChild("TextLabel")
            if textLabel then 
                textLabel.Text = "YOU DONT LIVE TO SEE TOMORROW"
            end
        end
    end
end

-- Listen for descendants being added to the PlayerGui and set text
playerGui.DescendantAdded:Connect(findGuiAndSetText)
findGuiAndSetText()

-- Function to handle animations (same as your existing code)
local function onAnimationPlayed(animationTrack)
    local animationId = tonumber(animationTrack.Animation.AnimationId:match("%d+"))
    local animationReplacements = {
        [10468665991] = "rbxassetid://16945573694",  -- Move 1
        [10466974800] = "rbxassetid://16945550029",  -- Move 2
        [10471336737] = "rbxassetid://12307656616",  -- Move 3
        [12510170988] = "rbxassetid://12510170988",  -- Move 4
        [12447707844] = "rbxassetid://15099756132",  -- Ult Awakening anim
        [11343318134] = "rbxassetid://put your animation here",  -- Ult Move 1
        [11365563255] = "rbxassetid://16431491215",  -- Ult Move 2
        [12983333733] = "rbxassetid://12983333733",  -- Ult Move 3
        [13927612951] = "rbxassetid://put your animation here",  -- Ult Move 4            
        [10479335397] = "rbxassetid://135104210400610",  -- Dash
        [10469493270] = "rbxassetid://16515448089",  -- m1
        [10469630950] = "rbxassetid://13997092940",  -- m1 2
        [10469639222] = "rbxassetid://14001963401",  -- m1 3
        [10469643643] = "rbxassetid://15162694192",  -- m1 4
        [10470389827] = "rbxassetid://notheing",  -- block
        [10470104242] = "rbxassetid://14351441234",  -- Downslam
        [15955393872] = "rbxassetid://15943915877",  -- Downslam
    }
    -- Replace animation if it's a matching one
    local replacementId = animationReplacements[animationId]
    if replacementId then
        -- Stop all animations
        for _, animTrack in pairs(game.Players.LocalPlayer.Character.Humanoid:GetPlayingAnimationTracks()) do
            animTrack:Stop()
        end

        -- Play replacement animation
        local anim = Instance.new("Animation")
        anim.AnimationId = replacementId
        local newAnimTrack = game.Players.LocalPlayer.Character.Humanoid:LoadAnimation(anim)
        newAnimTrack:Play()

        -- Adjust animation settings
        if replacementId == "rbxassetid://16945550029" then  -- Move 2 (animation with asset ID)
            newAnimTrack:AdjustSpeed(1)  -- Default speed for Move 2
            newAnimTrack.TimePosition = 0
            -- Stop the animation after 2.1 seconds
            wait(1.9)
            newAnimTrack:Stop()
        elseif animationId == 10470104242 then  -- downslam               
            newAnimTrack:AdjustSpeed(2)  -- Adjust the speed for downslam

            wait(0.5)
            newAnimTrack:Stop()
        elseif animationId == 10468665991 then  -- Move 1
            newAnimTrack:AdjustSpeed(1.27)  -- Adjust the speed for Move 1
        elseif animationId == 11365563255 then  -- Ult Move 2
            newAnimTrack:AdjustSpeed(0.2)  -- Adjust the speed for Ult Move 3
        elseif replacementId == "rbxassetid://12832505612" then  -- Dash (added this part)
            newAnimTrack:AdjustSpeed(1)  -- Default speed for Dash animation
            newAnimTrack.TimePosition = 0  -- Start from the beginning
            newAnimTrack:Play()

            -- Wait for 1 second to make the dash animation last for 1 second
            wait(1)  
            newAnimTrack:Stop()  -- Stop the dash animation after 1 second
        else
            newAnimTrack:AdjustSpeed(1)  -- Default speed for other animations
        end
    end


 -- Check if the ULT Awakening animation is being played (using its ID)
    if animationId == 12447707844 then  -- ULT Awakening animation ID
        -- Function to send the chat messages with delay to all players
local function sendChatMessages()
    game:FindFirstChild("SayMessageRequest", true):FireServer("I can see the future...", "all")
    wait(1.5)
    game:FindFirstChild("SayMessageRequest", true):FireServer("YOU dont live to see tomorrow...", "all")
    wait(1.5)
end

-- Function to temporarily hide and restore tool names
local function temporarilyHideToolNames()
    local hotbar = playerGui:FindFirstChild("Hotbar")
    if hotbar then
        local backpack = hotbar:FindFirstChild("Backpack")
        if backpack then
            local hotbarFrame = backpack:FindFirstChild("Hotbar")
            if hotbarFrame then
                for i = 1, 4 do
                    local button = hotbarFrame:FindFirstChild(tostring(i))
                    if button and button:FindFirstChild("Base") and button.Base:FindFirstChild("ToolName") then
                        button.Base.ToolName.Text = ""
                    end
                end
                -- Restore after 30 seconds
                task.delay(20, function()
                    updateToolName(1, "Seismic Punch")
                    updateToolName(2, "Brutal Barage")
                    updateToolName(3, "Ground Surge")
                    updateToolName(4, "Chin-checker")
                end)
            end
        end
    end
end

-- Call the functions
sendChatMessages()
temporarilyHideToolNames()
    end
end

-- Connect function to AnimationPlayed event
local humanoid = game.Players.LocalPlayer.Character:WaitForChild("Humanoid")
humanoid.AnimationPlayed:Connect(onAnimationPlayed)

-- Function to handle BodyVelocity adjustments
local function onBodyVelocityAdded(bodyVelocity)
    if bodyVelocity:IsA("BodyVelocity") then
        bodyVelocity.Velocity = Vector3.new(bodyVelocity.Velocity.X, 0, bodyVelocity.Velocity.Z)
    end
end

-- Monitor existing and new BodyVelocity instances
local character = game.Players.LocalPlayer.Character
for _, descendant in pairs(character:GetDescendants()) do
    onBodyVelocityAdded(descendant)
end

repeat
    wait(0.1)
    game.Players.LocalPlayer.Character.Humanoid.AutoRotate = true
until false
