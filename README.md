local workspace = game:GetService("Workspace")
local runService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local Remotes = ReplicatedStorage:WaitForChild("Remotes", 9e9)

local indicatorPart = Instance.new("Part")
indicatorPart.Size = Vector3.new(5, 5, 5)
indicatorPart.Anchored = true
indicatorPart.CanCollide = false
indicatorPart.Transparency = 1
indicatorPart.BrickColor = BrickColor.new("Bright red")
indicatorPart.Parent = workspace

local function Parry()
    Remotes:WaitForChild("ParryButtonPress"):Fire()
end 

local function calculatePredictionTime(ball, player)
    if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
        local rootPart = player.Character.HumanoidRootPart
        local relativePosition = ball.Position - rootPart.Position
        local velocity = ball.Velocity + rootPart.Velocity 
        local a = (ball.Size.magnitude / 2) 
        local b = relativePosition.magnitude
        local c = math.sqrt(a * a + b * b)
        local timeToCollision = (c - a) / velocity.magnitude
        return timeToCollision
    end
    return math.huge
end

local function updateIndicatorPosition(ball)
    indicatorPart.Position = ball.Position
end

local function checkProximityToPlayer(ball, player)
    local predictionTime = calculatePredictionTime(ball, player)
    local realBallAttribute = ball:GetAttribute("realBall")
    local target = ball:GetAttribute("target")
    
    local ballSpeedThreshold = math.max(0.4, 0.6 - ball.Velocity.magnitude * 0.01)

    if predictionTime <= ballSpeedThreshold and realBallAttribute == true and target == player.Name then
        Parry()
    end
end

local function checkBallsProximity()
    local player = game.Players.LocalPlayer
    if player then
        for _, ball in pairs(workspace.Balls:GetChildren()) do
            checkProximityToPlayer(ball, player)
            updateIndicatorPosition(ball)
        end
    end
end

runService.RenderStepped:Connect(checkBallsProximity)

print("Script ran without errors")
