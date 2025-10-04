
local player = game.Players.LocalPlayer

local screenGui = Instance.new("ScreenGui", player.PlayerGui)
screenGui.Name = "HitboxModGui"

local frame = Instance.new("Frame", screenGui)
frame.Size = UDim2.new(0, 270, 0, 150)
frame.Position = UDim2.new(0.5, -135, 0.3, 0)
frame.BackgroundColor3 = Color3.new(0.13, 0.13, 0.13)
frame.Active = true
frame.Draggable = true

local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1, 0, 0, 32)
title.Position = UDim2.new(0, 0, 0, 0)
title.Text = "Modificador de Hitbox de Oponentes"
title.BackgroundTransparency = 1
title.TextColor3 = Color3.new(1,1,1)
title.Font = Enum.Font.GothamBold
title.TextSize = 18

local textBox = Instance.new("TextBox", frame)
textBox.Size = UDim2.new(0.65, 0, 0, 32)
textBox.Position = UDim2.new(0.05, 0, 0.35, 0)
textBox.PlaceholderText = "Tamanho (ex: 5,5,5)"
textBox.Text = ""
textBox.TextColor3 = Color3.new(0,0,0)
textBox.Font = Enum.Font.Gotham
textBox.TextSize = 16

local activateButton = Instance.new("TextButton", frame)
activateButton.Size = UDim2.new(0.25, -5, 0, 32)
activateButton.Position = UDim2.new(0.72, 0, 0.35, 0)
activateButton.Text = "Ativar"
activateButton.BackgroundColor3 = Color3.new(0, 0.6, 0)
activateButton.TextColor3 = Color3.new(1,1,1)
activateButton.Font = Enum.Font.GothamBold
activateButton.TextSize = 16

local deactivateButton = Instance.new("TextButton", frame)
deactivateButton.Size = UDim2.new(0.9, 0, 0, 32)
deactivateButton.Position = UDim2.new(0.05, 0, 0.75, 0)
deactivateButton.Text = "Desativar"
deactivateButton.BackgroundColor3 = Color3.new(0.6, 0, 0)
deactivateButton.TextColor3 = Color3.new(1,1,1)
deactivateButton.Font = Enum.Font.GothamBold
deactivateButton.TextSize = 16

-- Função para buscar hitboxes dos oponentes
local function getOpponentHitboxes()
    local hitboxes = {}
    -- Para cada jogador, exceto você
    for _, plr in ipairs(game.Players:GetPlayers()) do
        if plr ~= player and plr.Character then
            for _, part in ipairs(plr.Character:GetChildren()) do
                if part:IsA("BasePart") then
                    table.insert(hitboxes, part)
                end
            end
            -- Ferramentas do personagem
            for _, tool in ipairs(plr.Character:GetChildren()) do
                if tool:IsA("Tool") then
                    for _, part in ipairs(tool:GetChildren()) do
                        if part:IsA("BasePart") then
                            table.insert(hitboxes, part)
                        end
                    end
                end
            end
            -- Ferramentas na mochila
            for _, tool in ipairs(plr.Backpack:GetChildren()) do
                if tool:IsA("Tool") then
                    for _, part in ipairs(tool:GetChildren()) do
                        if part:IsA("BasePart") then
                            table.insert(hitboxes, part)
                        end
                    end
                end
            end
        end
    end
    -- Bombas/explosivos
    for _, obj in ipairs(workspace:GetChildren()) do
        if obj.Name:lower():find("bomb") or obj.Name:lower():find("explosive") then
            if obj:IsA("BasePart") then
                table.insert(hitboxes, obj)
            end
        end
    end
    return hitboxes
end

-- Guardar tamanho original
local originalSizes = {}

-- Ativar
activateButton.MouseButton1Click:Connect(function()
    local sizeStr = textBox.Text
    local sizeVec
    -- Tenta converter para Vector3
    local x, y, z = sizeStr:match("(%d+%.?%d*),(%d+%.?%d*),(%d+%.?%d*)")
    if x and y and z then
        sizeVec = Vector3.new(tonumber(x), tonumber(y), tonumber(z))
    else
        local num = tonumber(sizeStr)
        if num then
            sizeVec = Vector3.new(num, num, num)
        else
            textBox.Text = "Formato inválido!"
            return
        end
    end

    for _, part in ipairs(getOpponentHitboxes()) do
        if not originalSizes[part] then
            originalSizes[part] = part.Size
        end
        part.Size = sizeVec
        part.CanCollide = true
        part.Transparency = 0.5
        part.Color = Color3.new(1, 0, 0)
    end
end)

-- Desativar
deactivateButton.MouseButton1Click:Connect(function()
    for part, size in pairs(originalSizes) do
        if part and part.Parent then
            part.Size = size
            part.Transparency = 0
            part.Color = Color3.new(1,1,1)
        end
    end
    originalSizes = {}
end)
