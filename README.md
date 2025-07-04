local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

function getRoleColor(player)
    if player.Character then
        if player.Character:FindFirstChild("Knife") then
            return Color3.fromRGB(255, 0, 0) -- Assassino
        elseif player.Backpack:FindFirstChild("Gun") or player.Character:FindFirstChild("Gun") then
            return Color3.fromRGB(0, 0, 255) -- Xerife
        end
    end
    return Color3.fromRGB(255, 255, 0) -- Inocente
end

function createESP(player)
    if player == LocalPlayer then return end

    local texto = Drawing.new("Text")
    texto.Text = "ENEMY"
    texto.Size = 14
    texto.Center = true
    texto.Outline = true
    texto.Visible = true
    texto.Color = getRoleColor(player)

    RunService.RenderStepped:Connect(function()
        pcall(function()
            if not player.Character or not player.Character:FindFirstChild("Head") then
                texto.Visible = false
                return
            end

            local pos, vis = Camera:WorldToViewportPoint(player.Character.Head.Position + Vector3.new(0, 0.5, 0))
            if vis and player.Character:FindFirstChild("Humanoid") and player.Character.Humanoid.Health > 0 then
                texto.Position = Vector2.new(pos.X, pos.Y)
                texto.Color = getRoleColor(player)
                texto.Visible = true
            else
                texto.Visible = false
            end
        end)
    end)
end

-- Adiciona ESP para quem já está no jogo
for _, player in ipairs(Players:GetPlayers()) do
    createESP(player)
end

-- Adiciona ESP para quem entrar depois
Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function()
        wait(1)
        createESP(player)
    end)
end)
