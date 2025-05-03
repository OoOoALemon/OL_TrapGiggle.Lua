local Player = game.Players.LocalPlayer
local Character = Player.Character or Player.CharacterAdded:Wait()
local Humanoid = Character:FindFirstChild("Humanoid")

local Tool = game:GetObjects("rbxassetid://135573899162791")[1]
Tool.Parent = Player.Backpack
Tool.RequiresHandle = true

local Sounds = Instance.new("Folder")
Sounds.Name = "Sounds"
Sounds.Parent = Tool

local soundIds = {
    "rbxassetid://17740288597",
    "rbxassetid://17740288342",
    "rbxassetid://17740288225",
    "rbxassetid://17740288457"
}

local soundInstances = {}
for _, soundId in ipairs(soundIds) do
    local sound = Instance.new("Sound")
    sound.SoundId = soundId
    sound.Parent = Sounds
    table.insert(soundInstances, sound)
end

local ragdollTemplate = game:GetObjects("rbxassetid://78653424928963")[1]
ragdollTemplate.Parent = game.ReplicatedStorage

local mainPart = ragdollTemplate:FindFirstChild("Main")
if mainPart and mainPart:IsA("MeshPart") then
    ragdollTemplate.PrimaryPart = mainPart
end

Tool.Activated:Connect(function()
    local position = Character.HumanoidRootPart.Position
    local orientation = Character.HumanoidRootPart.CFrame.LookVector

    local ragdoll = ragdollTemplate:Clone()
    ragdoll.Parent = game.Workspace
    ragdoll:SetPrimaryPartCFrame(CFrame.new(position) * CFrame.new(0, 1, 0))

    local ragdollMainPart = ragdoll:FindFirstChild("Main")
    if ragdollMainPart and ragdollMainPart:IsA("MeshPart") then
        ragdoll.PrimaryPart = ragdollMainPart
    end

    local bodyVelocity = Instance.new("BodyVelocity")
    bodyVelocity.Velocity = orientation * 100
    bodyVelocity.MaxForce = Vector3.new(1e5, 1e5, 1e5)
    bodyVelocity.Parent = ragdoll.PrimaryPart

    local randomSound = soundInstances[math.random(#soundInstances)]
    randomSound:Play()

    local touched = false

    local function onAnyTouch(hit)
        if touched then return end

        for _, player in pairs(game.Players:GetPlayers()) do
            if player.Character and hit:IsDescendantOf(player.Character) then
                return
            end
        end

        if hit:IsDescendantOf(Tool) then
            return
        end

        if hit.Transparency == 1 then
            return
        end

        if hit:IsA("BasePart") then
            touched = true

            for _, obj in ipairs(hit:GetDescendants()) do
                if obj:IsA("Decal") or obj:IsA("Texture") or obj:IsA("SurfaceAppearance") then
                    obj:Destroy()
                end
            end

            if hit:IsA("BasePart") then
                hit.MaterialVariant = nil
                hit.Color = Color3.fromRGB(200, 200, 200)
                hit.Material = Enum.Material.Salt
            end

            if ragdoll and ragdoll.Parent then
                ragdoll:Destroy()
            end
        end
    end

    for _, part in ipairs(ragdoll:GetDescendants()) do
        if part:IsA("BasePart") then
            part.Touched:Connect(onAnyTouch)
        end
    end

    local timeout = 10
    local timer = tick()
    game:GetService("RunService").Heartbeat:Connect(function()
        if tick() - timer >= timeout then
            if ragdoll and ragdoll.Parent then
                ragdoll:Destroy()
            end
        end
    end)
end)

local Animation = Instance.new("Animation")
Animation.AnimationId = "rbxassetid://10479585177"
Animation.Name = "Idle"

local Idle

Tool.Equipped:Connect(function()
    if Humanoid and Humanoid:FindFirstChild("Animator") then
        Idle = Humanoid.Animator:LoadAnimation(Animation)
        Idle.Looped = true
        Idle:Play()
    end
end)

Tool.Unequipped:Connect(function()
    if Idle then
        Idle:Stop()
        Idle = nil
    end
end)
