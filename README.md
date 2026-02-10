loadstring(game:HttpGet("https://raw.githubusercontent.com/me2v/Scripts-roblox/refs/heads/main/README.md"))()

-- [[ CARREGAMENTO DA RAYFIELD ]]
local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
    Name = "FFH4X 💀 V10",
    LoadingTitle = "INICIALIZANDO SISTEMA HAXWX...",
    LoadingSubtitle = "O Script Mais Completo para FPS",
    ConfigurationSaving = { Enabled = true, FolderName = "FFH4X_Haxwx" },
    KeySystem = false 
})

-- [[ CONFIGURAÇÕES DO SCRIPT ]]
getgenv().Config = {
    -- Combate
    Aimbot = false,
    Smoothness = 2,
    FOV = 100, -- FOV do Círculo do Aimbot
    WallCheck = true,
    TargetPart = "Head",
    HitboxSize = 2,
    -- Visuals
    ESP_Box = false,
    StreamerMode = false,
    -- Tela Esticada (Matrix)
    StretchRes = false,
    StretchAmount = 0.6, -- Quanto menor, mais esticado (gordo)
    -- Movimento
    Speed = 16,
    Fly = false,
    Noclip = false,
    InfiniteJump = false,
    CleanMovement = false
}

-- [[ FOV VISUAL DO AIMBOT ]]
local FOVCircle = Drawing.new("Circle")
FOVCircle.Thickness = 1.5
FOVCircle.Color = Color3.fromRGB(255, 0, 0)
FOVCircle.Filled = false
FOVCircle.Transparency = 1
FOVCircle.Visible = true

-- ==========================================
-- [ABA 1: COMBATE ⚔️]
-- ==========================================
local CombatTab = Window:CreateTab("Combate 🎯")

CombatTab:CreateToggle({ Name = "Aimbot Lock (Segurar Click)", CurrentValue = false, Callback = function(v) getgenv().Config.Aimbot = v end, })
CombatTab:CreateSlider({ Name = "Suavização (0 = Rage | 10 = Legit)", Range = {0, 10}, Increment = 1, CurrentValue = 2, Callback = function(v) getgenv().Config.Smoothness = v end, })
CombatTab:CreateSlider({ Name = "Raio do FOV (Aimbot)", Range = {0, 600}, Increment = 10, CurrentValue = 100, Callback = function(v) getgenv().Config.FOV = v end, })
CombatTab:CreateToggle({ Name = "Wall Check (Não atravessa parede)", CurrentValue = true, Callback = function(v) getgenv().Config.WallCheck = v end, })
CombatTab:CreateSlider({ Name = "Expandir Hitbox Inimiga", Range = {2, 25}, Increment = 1, CurrentValue = 2, Callback = function(v) getgenv().Config.HitboxSize = v end, })

-- ==========================================
-- [ABA 2: VISUALS 👁️]
-- ==========================================
local VisualTab = Window:CreateTab("Visuals 👁️")

VisualTab:CreateToggle({ Name = "ESP Box (Time + Vida)", CurrentValue = false, Callback = function(v) getgenv().Config.ESP_Box = v end, })
VisualTab:CreateToggle({ Name = "Modo Streamer (Esconde Tudo)", CurrentValue = false, Callback = function(v) getgenv().Config.StreamerMode = v FOVCircle.Visible = not v end, })

VisualTab:CreateSection("Tela Esticada (Resolução)")
VisualTab:CreateLabel("Faz os bonecos ficarem largos (Gordinhos)")

VisualTab:CreateToggle({
    Name = "Ativar Resolução Esticada (Matrix)",
    CurrentValue = false,
    Callback = function(v)
        getgenv().Config.StretchRes = v
    end,
})

VisualTab:CreateSlider({
    Name = "Intensidade (Menor = Mais Largo)",
    Range = {0.3, 1.0}, -- 1.0 é normal, 0.5 é bem esticado
    Increment = 0.05,
    CurrentValue = 0.6,
    Callback = function(v)
        getgenv().Config.StretchAmount = v
    end,
})

VisualTab:CreateKeybind({ Name = "Panic Button (P)", CurrentKeybind = "P", HoldToInteract = false, Callback = function() local gui = game.CoreGui:FindFirstChild("Rayfield") if gui then gui.Enabled = not gui.Enabled end getgenv().Config.StreamerMode = not getgenv().Config.StreamerMode FOVCircle.Visible = not getgenv().Config.StreamerMode end, })

-- ==========================================
-- [ABA 3: MOVIMENTO 🏃]
-- ==========================================
local MoveTab = Window:CreateTab("Movimento 🏃")

MoveTab:CreateSlider({ Name = "Velocidade (Max 160)", Range = {16, 160}, Increment = 1, CurrentValue = 16, Callback = function(v) getgenv().Config.Speed = v end, })
MoveTab:CreateToggle({ Name = "Voo Perfeito + Noclip", CurrentValue = false, Callback = function(v) getgenv().Config.Fly = v; getgenv().Config.Noclip = v end, })
MoveTab:CreateToggle({ Name = "Pulo Infinito", CurrentValue = false, Callback = function(v) getgenv().Config.InfiniteJump = v end, })

-- ==========================================
-- [[ LÓGICA DO SISTEMA ]]
-- ==========================================
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local LP = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- Verificação de Visibilidade
local function IsVisible(part)
    local params = RaycastParams.new()
    params.FilterType = Enum.RaycastFilterType.Exclude
    params.FilterDescendantsInstances = {LP.Character, Camera}
    local res = workspace:Raycast(Camera.CFrame.Position, (part.Position - Camera.CFrame.Position), params)
    return res == nil or res.Instance:IsDescendantOf(part.Parent)
end

-- Busca de Alvo
local function GetTarget()
    local target, shortestDist = nil, getgenv().Config.FOV
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= LP and p.Team ~= LP.Team and p.Character and p.Character:FindFirstChild("Head") and p.Character.Humanoid.Health > 0 then
            local pos, vis = Camera:WorldToViewportPoint(p.Character.Head.Position)
            if vis then
                local dist = (Vector2.new(pos.X, pos.Y) - Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)).Magnitude
                if dist < shortestDist then
                    if not getgenv().Config.WallCheck or IsVisible(p.Character.Head) then
                        target = p
                        shortestDist = dist
                    end
                end
            end
        end
    end
    return target
end

-- Pulo Infinito
UIS.JumpRequest:Connect(function()
    if getgenv().Config.InfiniteJump and LP.Character then
        LP.Character:FindFirstChildOfClass("Humanoid"):ChangeState("Jumping")
    end
end)

-- [[ SISTEMA DE TELA ESTICADA (MATRIX) ]]
-- Usa alta prioridade para modificar a câmera logo antes da renderização
RunService:BindToRenderStep("Haxwx_Stretch", Enum.RenderPriority.Camera.Value + 1, function()
    if getgenv().Config.StretchRes then
        -- Multiplica a CFrame da Câmera por uma matriz que achata o eixo Y
        -- (1,0,0) = X normal, (0, Y, 0) = Y achatado, (0,0,1) = Z normal
        Camera.CFrame = Camera.CFrame * CFrame.new(0, 0, 0, 1, 0, 0, 0, getgenv().Config.StretchAmount, 0, 0, 0, 1)
    end
end)

-- Loop Geral (Aimbot, ESP, Movimento)
RunService.RenderStepped:Connect(function()
    -- Atualiza FOV Aim (Círculo)
    FOVCircle.Radius = getgenv().Config.FOV
    FOVCircle.Position = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)

    -- Aimbot Suave/Rage
    if getgenv().Config.Aimbot then
        local t = GetTarget()
        if t and UIS:IsMouseButtonPressed(Enum.UserInputType.MouseButton1) then
            local targetPos = t.Character.Head.Position
            if getgenv().Config.Smoothness == 0 then
                Camera.CFrame = CFrame.new(Camera.CFrame.Position, targetPos)
            else
                local smooth = (11 - getgenv().Config.Smoothness) * 0.05
                -- Nota: A interpolação (Lerp) pode brigar um pouco com o Stretch, mas costuma funcionar.
                Camera.CFrame = Camera.CFrame:Lerp(CFrame.new(Camera.CFrame.Position, targetPos), smooth)
            end
        end
    end

    -- ESP, Hitbox e Movimento
    if LP.Character and LP.Character:FindFirstChild("Humanoid") then
        LP.Character.Humanoid.WalkSpeed = getgenv().Config.Speed

        -- Fly
        if getgenv().Config.Fly then
            LP.Character.HumanoidRootPart.Velocity = Vector3.new(0, 1.5, 0)
        end

        -- Noclip
        if getgenv().Config.Noclip then
            for _, v in pairs(LP.Character:GetDescendants()) do
                if v:IsA("BasePart") then v.CanCollide = false end
            end
        end

        -- Loop Jogadores
        for _, p in pairs(Players:GetPlayers()) do
            if p ~= LP and p.Character and p.Character:FindFirstChild("Humanoid") then
                -- Hitbox
                local hrp = p.Character:FindFirstChild("HumanoidRootPart")
                if hrp and p.Team ~= LP.Team then
                    hrp.Size = Vector3.new(getgenv().Config.HitboxSize, getgenv().Config.HitboxSize, getgenv().Config.HitboxSize)
                    hrp.Transparency = (getgenv().Config.HitboxSize > 2) and 0.7 or 1
                end

                -- ESP
                local h = p.Character:FindFirstChild("Haxwx_ESP")
                if getgenv().Config.ESP_Box and not getgenv().Config.StreamerMode then
                    if not h then h = Instance.new("Highlight", p.Character); h.Name = "Haxwx_ESP"; h.FillTransparency = 1 end
                    h.OutlineColor = (p.Team == LP.Team) and Color3.new(0,1,0) or Color3.new(1,0,0)
                    p.Character.Humanoid.DisplayName = "HP: " .. math.floor(p.Character.Humanoid.Health)
                elseif h then
                    h:Destroy()
                end
            end
        end
    end
end)
