--[[ 
    LZhin_Fly_Xeno - Versão Universal
    Compatível com: Xeno, Solara, Wave, Electron, JJSploit, etc.
]]

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "LZhin_Fly_Xeno"
ScreenGui.ResetOnSpawn = false -- Mantém a GUI ao morrer

-- Proteção de Parent (Universal)
local function getHidingPart()
    local success, core = pcall(function() return game:GetService("CoreGui") end)
    if success and core then return core end
    return game:GetService("Players").LocalPlayer:WaitForChild("PlayerGui")
end
ScreenGui.Parent = getHidingPart()

local player = game.Players.LocalPlayer
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local userId = player.UserId

-- 1. CARREGAMENTO
local LoadingFrame = Instance.new("Frame")
LoadingFrame.Size = UDim2.new(0, 200, 0, 100)
LoadingFrame.Position = UDim2.new(0.5, -100, 0.5, -50)
LoadingFrame.BackgroundColor3 = Color3.new(0, 0, 0)
LoadingFrame.Parent = ScreenGui
local corner = Instance.new("UICorner", LoadingFrame)

local lt = Instance.new("TextLabel", LoadingFrame)
lt.Size = UDim2.new(1, 0, 1, 0)
lt.Text = "Carregando LZhin..."
lt.TextColor3 = Color3.new(1, 1, 1)
lt.BackgroundTransparency = 1
lt.Font = Enum.Font.SourceSansBold
lt.TextSize = 20

-- 2. EMBLEMA (Miguelsf1804)
local function ShowBadge()
    local Badge = Instance.new("Frame", ScreenGui)
    Badge.Size = UDim2.new(0, 220, 0, 60)
    Badge.Position = UDim2.new(0.5, -110, -0.2, 0)
    Badge.BackgroundColor3 = Color3.new(0, 0, 0)
    Instance.new("UICorner", Badge)
    Instance.new("UIStroke", Badge).Color = Color3.new(1,1,1)
    
    local img = Instance.new("ImageLabel", Badge)
    img.Size = UDim2.new(0, 45, 0, 45)
    img.Position = UDim2.new(0, 8, 0.5, -22)
    img.Image = "rbxthumb://type=AvatarHeadShot&id="..userId.."&w=150&h=150"
    Instance.new("UICorner", img).CornerRadius = UDim.new(1, 0)
    
    local txt = Instance.new("TextLabel", Badge)
    txt.Size = UDim2.new(0, 150, 1, 0)
    txt.Position = UDim2.new(0, 60, 0, 0)
    txt.Text = "Eae mano(a) 👋"
    txt.TextColor3 = Color3.new(1, 1, 1)
    txt.BackgroundTransparency = 1
    txt.Font = Enum.Font.SourceSansBold
    txt.TextSize = 16
    txt.TextXAlignment = Enum.TextXAlignment.Left

    Badge:TweenPosition(UDim2.new(0.5, -110, 0.05, 0), "Out", "Quad", 0.5, true)
    task.wait(3)
    Badge:TweenPosition(UDim2.new(0.5, -110, -0.2, 0), "In", "Quad", 0.5, true)
    task.wait(0.6)
    Badge:Destroy()
end

task.wait(1)
LoadingFrame:Destroy()
task.spawn(ShowBadge)

-- 3. MENU PRINCIPAL
local MainFrame = Instance.new("Frame", ScreenGui)
MainFrame.BackgroundColor3 = Color3.new(0, 0, 0)
MainFrame.Position = UDim2.new(0.5, -100, 0.5, -75)
MainFrame.Size = UDim2.new(0, 200, 0, 150)
MainFrame.Active = true
MainFrame.Draggable = true -- Nota: Alguns executores podem precisar de script de drag customizado
Instance.new("UICorner", MainFrame)
Instance.new("UIStroke", MainFrame).Color = Color3.new(1,1,1)

local Title = Instance.new("TextLabel", MainFrame)
Title.Text = "LZhin ✅ (Universal)"
Title.TextColor3 = Color3.new(1,1,1)
Title.Size = UDim2.new(1, 0, 0, 30)
Title.BackgroundTransparency = 1
Title.Font = Enum.Font.SourceSansBold
Title.TextSize = 18

local FlyBtn = Instance.new("TextButton", MainFrame)
FlyBtn.Position = UDim2.new(0.1, 0, 0.35, 0)
FlyBtn.Size = UDim2.new(0.8, 0, 0, 30)
FlyBtn.Text = "Fly & God: OFF"
FlyBtn.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
FlyBtn.TextColor3 = Color3.new(1,1,1)
Instance.new("UICorner", FlyBtn)

local SpeedSlider = Instance.new("TextBox", MainFrame)
SpeedSlider.Position = UDim2.new(0.1, 0, 0.65, 0)
SpeedSlider.Size = UDim2.new(0.8, 0, 0, 25)
SpeedSlider.Text = "50"
SpeedSlider.PlaceholderText = "Velocidade"
SpeedSlider.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
SpeedSlider.TextColor3 = Color3.new(1,1,1)
Instance.new("UICorner", SpeedSlider)

local CloseBtn = Instance.new("TextButton", MainFrame)
CloseBtn.Position = UDim2.new(0.85, 0, 0, 5)
CloseBtn.Size = UDim2.new(0, 20, 0, 20)
CloseBtn.Text = "X"
CloseBtn.BackgroundColor3 = Color3.fromRGB(150, 0, 0)
CloseBtn.TextColor3 = Color3.new(1,1,1)
Instance.new("UICorner", CloseBtn)

local OpenBtn = Instance.new("TextButton", ScreenGui)
OpenBtn.Visible = false
OpenBtn.Position = UDim2.new(0, 10, 0.5, -15)
OpenBtn.Size = UDim2.new(0, 60, 0, 30)
OpenBtn.Text = "Abrir"
OpenBtn.BackgroundColor3 = Color3.new(0,0,0)
OpenBtn.TextColor3 = Color3.new(1,1,1)
Instance.new("UIStroke", OpenBtn).Color = Color3.new(1,1,1)
Instance.new("UICorner", OpenBtn)

-- 4. LÓGICA DE MOVIMENTO (REFEITA PARA ESTABILIDADE)
local flying = false
local speed = 50
local bv, bg, ff

local function cleanup()
    if bv then bv:Destroy(); bv = nil end
    if bg then bg:Destroy(); bg = nil end
    if ff then ff:Destroy(); ff = nil end
    local hum = player.Character and player.Character:FindFirstChildOfClass("Humanoid")
    if hum then hum.PlatformStand = false end
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
    
    if hrp and hum then
        -- God Mode (ForceField + Loop de Vida)
        ff = Instance.new("ForceField", char)
        ff.Visible = false
        
        -- Configuração de Física
        bg = Instance.new("BodyGyro")
        bg.P = 9e4
        bg.maxTorque = Vector3.new(9e9, 9e9, 9e9)
        bg.cframe = hrp.CFrame
        bg.Parent = hrp
        
        bv = Instance.new("BodyVelocity")
        bv.velocity = Vector3.new(0,0,0)
        bv.maxForce = Vector3.new(9e9, 9e9, 9e9)
        bv.Parent = hrp
        
        task.spawn(function()
            while flying and char and char.Parent do
                local dt = RunService.RenderStepped:Wait()
                local cam = workspace.CurrentCamera.CFrame
                local moveDir = Vector3.new(0,0,0)
                
                speed = tonumber(SpeedSlider.Text) or 50
                hum.PlatformStand = true -- Evita animação de queda
                hum.Health = hum.MaxHealth -- Auto-heal
                
                if UIS:IsKeyDown(Enum.KeyCode.W) then moveDir = moveDir + cam.LookVector end
                if UIS:IsKeyDown(Enum.KeyCode.S) then moveDir = moveDir - cam.LookVector end
                if UIS:IsKeyDown(Enum.KeyCode.A) then moveDir = moveDir - cam.RightVector end
                if UIS:IsKeyDown(Enum.KeyCode.D) then moveDir = moveDir + cam.RightVector end
                if UIS:IsKeyDown(Enum.KeyCode.Space) then moveDir = moveDir + Vector3.new(0,1,0) end
                if UIS:IsKeyDown(Enum.KeyCode.LeftShift) then moveDir = moveDir - Vector3.new(0,1,0) end
                
                if moveDir.Magnitude > 0 then
                    bv.velocity = moveDir.Unit * speed
                else
                    bv.velocity = Vector3.new(0, 0.1, 0)
                end
                bg.cframe = cam
            end
            cleanup()
        end)
    end
end

-- Conexões
FlyBtn.MouseButton1Click:Connect(toggleFly)
CloseBtn.MouseButton1Click:Connect(function() MainFrame.Visible = false; OpenBtn.Visible = true end)
OpenBtn.MouseButton1Click:Connect(function() MainFrame.Visible = true; OpenBtn.Visible = false end)

UIS.InputBegan:Connect(function(input, gpe)
    if not gpe and input.KeyCode == Enum.KeyCode.F then
        toggleFly()
    end
end)

-- Re-aplicar se o personagem resetar
player.CharacterAdded:Connect(function()
    if flying then
        task.wait(0.5)
        flying = false
        toggleFly()
    end
end)
