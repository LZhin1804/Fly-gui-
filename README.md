-- 4. LÓGICA DE MOVIMENTO (ESTILO IY COM CORREÇÃO DE BUG NO CHÃO)
local flying = false
local speed = 50
local ctrl = {f = 0, b = 0, l = 0, r = 0}

local function cleanup()
    local char = player.Character
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    
    if hum then 
        hum.PlatformStand = false 
        -- Força o estado de "Landed" (Pousado) para o jogo entender que você não está mais caindo/voando
        hum:ChangeState(Enum.HumanoidStateType.Landing)
    end
    
    if hrp then
        -- Remove TODAS as forças de movimento criadas pelo script
        for _, v in pairs(hrp:GetChildren()) do
            if v:IsA("BodyGyro") or v:IsA("BodyVelocity") or v:IsA("BodyPosition") then 
                v:Destroy() 
            end
        end
        -- Zera a velocidade residual para o personagem não sair "deslizando"
        hrp.Velocity = Vector3.new(0, 0, 0)
        hrp.RotVelocity = Vector3.new(0, 0, 0)
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
        -- O segredo do IY: Criar as forças e mantê-las controladas
        local bv = Instance.new("BodyVelocity")
        bv.Name = "LY_Fly_BV"
        bv.MaxForce = Vector3.new(9e9, 9e9, 9e9)
        bv.Velocity = Vector3.new(0, 0, 0)
        bv.Parent = hrp
        
        local bg = Instance.new("BodyGyro")
        bg.Name = "LY_Fly_BG"
        bg.MaxTorque = Vector3.new(9e9, 9e9, 9e9)
        bg.P = 9e4
        bg.CFrame = hrp.CFrame
        bg.Parent = hrp

        task.spawn(function()
            while flying and char and char.Parent do
                local dt = RunService.RenderStepped:Wait()
                speed = tonumber(SpeedSlider.Text) or 50
                
                hum.PlatformStand = true -- Mantém o personagem em modo "plataforma" (estático)
                
                if ctrl.l + ctrl.r ~= 0 or ctrl.f + ctrl.b ~= 0 then
                    bv.Velocity = ((camera.CFrame.LookVector * (ctrl.f + ctrl.b)) + ((camera.CFrame * CFrame.new(ctrl.l + ctrl.r, (ctrl.f + ctrl.b) * 0.2, 0).p) - camera.CFrame.p)) * speed
                else
                    bv.Velocity = Vector3.new(0, 0.1, 0) -- Pequena força para cima para ignorar a gravidade
                end
                
                bg.CFrame = camera.CFrame
            end
            cleanup() -- Garante a limpeza se o loop quebrar
        end)
    end
end
