-- 4. LÓGICA DE MOVIMENTO (ESTILO INFINITE YIELD - CFRAME)
local flying = false
local speed = 50
local ctrl = {f = 0, b = 0, l = 0, r = 0}
local lastctrl = {f = 0, b = 0, l = 0, r = 0}

local function cleanup()
    local char = player.Character
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    if hum then 
        hum.PlatformStand = false 
    end
    -- Remove forças residuais
    if char and char:FindFirstChild("HumanoidRootPart") then
        for _, v in pairs(char.HumanoidRootPart:GetChildren()) do
            if v:IsA("BodyGyro") or v:IsA("BodyVelocity") then v:Destroy() end
        end
    end
end

function toggleFly()
    flying = not flying
    FlyBtn.Text = flying and "Fly: ON" or "Fly: OFF"
    FlyBtn.TextColor3 = flying and Color3.new(0,1,0) or Color3.new(1,1,1)
    
    if not flying then
        cleanup()
        return
    end

    local char = player.Character
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    local camera = workspace.CurrentCamera
    
    if hrp and hum then
        hum.PlatformStand = true
        
        -- O segredo do IY: Um BodyVelocity zerado para anular a gravidade
        local bv = Instance.new("BodyVelocity", hrp)
        bv.MaxForce = Vector3.new(9e9, 9e9, 9e9)
        bv.Velocity = Vector3.new(0, 0, 0)
        
        local bg = Instance.new("BodyGyro", hrp)
        bg.MaxTorque = Vector3.new(9e9, 9e9, 9e9)
        bg.P = 9e4
        bg.CFrame = hrp.CFrame

        task.spawn(function()
            repeat
                RunService.RenderStepped:Wait()
                speed = tonumber(SpeedSlider.Text) or 50
                hum.PlatformStand = true
                
                -- Detecção de teclas estilo IY
                if ctrl.l + ctrl.r ~= 0 or ctrl.f + ctrl.b ~= 0 then
                    bv.Velocity = ((camera.CFrame.LookVector * (ctrl.f + ctrl.b)) + ((camera.CFrame * CFrame.new(ctrl.l + ctrl.r, (ctrl.f + ctrl.b) * 0.2, 0).p) - camera.CFrame.p)) * speed
                    lastctrl = {f = ctrl.f, b = ctrl.b, l = ctrl.l, r = ctrl.r}
                elseif speed == 0 then
                    bv.Velocity = Vector3.new(0, 0, 0)
                else
                    bv.Velocity = Vector3.new(0, 0.1, 0)
                end
                
                bg.CFrame = camera.CFrame
            until not flying
            cleanup()
        end)
    end
end

-- Atualização de Controles (Necessário para o estilo IY)
UIS.InputBegan:Connect(function(input, gpe)
    if gpe then return end
    if input.KeyCode == Enum.KeyCode.W then ctrl.f = 1
    elseif input.KeyCode == Enum.KeyCode.S then ctrl.b = -1
    elseif input.KeyCode == Enum.KeyCode.A then ctrl.l = -1
    elseif input.KeyCode == Enum.KeyCode.D then ctrl.r = 1
    elseif input.KeyCode == Enum.KeyCode.F then toggleFly() end
end)

UIS.InputEnded:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.W then ctrl.f = 0
    elseif input.KeyCode == Enum.KeyCode.S then ctrl.b = 0
    elseif input.KeyCode == Enum.KeyCode.A then ctrl.l = 0
    elseif input.KeyCode == Enum.KeyCode.D then ctrl.r = 0 end
end)
