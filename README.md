--// Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

--// Estado e UI
local ativo = true
local marcadores = {}

-- Criar botão
local gui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
gui.ResetOnSpawn = false
local botao = Instance.new("TextButton", gui)
botao.Size = UDim2.new(0, 100, 0, 40)
botao.Position = UDim2.new(0, 10, 0, 60)
botao.Text = "ESP ON"
botao.BackgroundColor3 = Color3.fromRGB(0, 200, 0)
botao.TextColor3 = Color3.new(1, 1, 1)
botao.TextScaled = true

botao.MouseButton1Click:Connect(function()
    ativo = not ativo
    botao.Text = ativo and "ESP ON" or "ESP OFF"
    botao.BackgroundColor3 = ativo and Color3.fromRGB(0, 200, 0) or Color3.fromRGB(200, 0, 0)
    for _, d in pairs(marcadores) do
        d.Text.Visible = ativo
    end
end)

--// Criar marcador
function criarMarcador(objeto, textoLabel, cor)
    local texto = Drawing.new("Text")
    texto.Text = textoLabel
    texto.Size = 14
    texto.Center = true
    texto.Outline = true
    texto.Color = cor
    texto.Visible = true

    RunService.RenderStepped:Connect(function()
        pcall(function()
            if objeto and objeto:IsDescendantOf(workspace) then
                local pos, vis = Camera:WorldToViewportPoint(objeto.Position)
                if vis and ativo then
                    texto.Position = Vector2.new(pos.X, pos.Y)
                    texto.Visible = true
                else
                    texto.Visible = false
                end
            else
                texto.Visible = false
            end
        end)
    end)

    table.insert(marcadores, {Text = texto})
end

--// Jogadores (Marretão / Sobreviventes)
for _, p in pairs(Players:GetPlayers()) do
    if p ~= LocalPlayer then
        p.CharacterAdded:Connect(function(char)
            wait(1)
            local cor = Color3.fromRGB(255, 255, 0) -- amarelo (inocente)
            if char:FindFirstChild("Hammer") then
                cor = Color3.fromRGB(255, 0, 0) -- vermelho (marretão)
            end
            if char:FindFirstChild("Head") then
                criarMarcador(char.Head, "PLAYER", cor)
            end
        end)
        if p.Character and p.Character:FindFirstChild("Head") then
            local cor = p.Character:FindFirstChild("Hammer") and Color3.fromRGB(255, 0, 0) or Color3.fromRGB(255, 255, 0)
            criarMarcador(p.Character.Head, "PLAYER", cor)
        end
    end
end

--// Computadores
for _, v in pairs(workspace:GetDescendants()) do
    if v.Name == "ComputerTable" and v:IsA("Model") then
        if v:FindFirstChild("ComputerScreen") then
            criarMarcador(v.ComputerScreen, "💻 PC", Color3.fromRGB(0, 255, 255))
        end
    end
end

--// Portas de saída
for _, v in pairs(workspace:GetDescendants()) do
    if v.Name == "ExitDoor" and v:IsA("Part") then
        criarMarcador(v, "🚪 SAÍDA", Color3.fromRGB(0, 255, 0))
    end
end

--// Câpsulas
for _, v in pairs(workspace:GetDescendants()) do
    if v.Name == "Tube" and v:IsA("Model") and v:FindFirstChild("Glass") then
        criarMarcador(v.Glass, "📦 CÁPSULA", Color3.fromRGB(150, 0, 255))
    end
end
