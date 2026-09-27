-- TROPA DO LKK — Premium Edition v6.6.0 (FIXED)
-- Corrige: Enum direto, perf de veículos, anti-lag travando, restore de Sky,
-- Neck em R6, WS restore no respawn, leaks de ESP/Contador, remotes especulativos
-- removidos (servidor push-only — ver dossiê).

local Players            = game:GetService("Players")
local RunService         = game:GetService("RunService")
local UserInputService   = game:GetService("UserInputService")
local TweenService       = game:GetService("TweenService")
local ReplicatedStorage  = game:GetService("ReplicatedStorage")
local HttpService        = game:GetService("HttpService")
local Workspace          = game:GetService("Workspace")
local Lighting           = game:GetService("Lighting")
local LocalPlayer        = Players.LocalPlayer
local Camera             = workspace.CurrentCamera
local PlayerGui          = LocalPlayer:WaitForChild("PlayerGui")

local VIM = nil
pcall(function() VIM = game:GetService("VirtualInputManager") end)

-- ===================== SAFE HELPERS =====================
local KC_CACHE = {}
local function SafeKeyCode(name)
    if type(name) ~= "string" then return nil end
    if KC_CACHE[name] ~= nil then return KC_CACHE[name] or nil end
    local ok, kc = pcall(function() return Enum.KeyCode[name] end)
    KC_CACHE[name] = ok and kc or false
    return ok and kc or nil
end

local K_UP    = SafeKeyCode("Up")
local K_DOWN  = SafeKeyCode("Down")
local K_HOME  = SafeKeyCode("Home")
local K_F     = SafeKeyCode("F")
local K_LCTRL = SafeKeyCode("LeftControl")
local K_W     = SafeKeyCode("W")
local K_S     = SafeKeyCode("S")
local K_A     = SafeKeyCode("A")
local K_D     = SafeKeyCode("D")
local K_SPACE = SafeKeyCode("Space")

local function FireMouse1Press()
    if mouse1press then pcall(mouse1press)
    elseif VIM then pcall(function() VIM:SendMouseButtonEvent(0,0,0,true,game,1) end) end
end
local function FireMouse1Release()
    if mouse1release then pcall(mouse1release)
    elseif VIM then pcall(function() VIM:SendMouseButtonEvent(0,0,0,false,game,1) end) end
end

local RestoreStack = setmetatable({}, {__mode = "k"})
local function SaveOrig(key, val) if key ~= nil and RestoreStack[key] == nil then RestoreStack[key] = val end end

local AutoDisableTimers = { noclip=30, underground=15, colar=20, giantgun=60 }

-- ===================== PALETA =====================
local BG, BG2, SIDE_BG, SIDE_BG_TOP = Color3.fromRGB(10,10,16), Color3.fromRGB(16,16,24), Color3.fromRGB(16,16,22), Color3.fromRGB(24,24,34)
local CARD_BG, CARD_HOVER, INPUT_BG, BORDER = Color3.fromRGB(26,26,36), Color3.fromRGB(42,42,56), Color3.fromRGB(38,38,50), Color3.fromRGB(80,80,105)
local TXT, TXT_DIM = Color3.fromRGB(255,255,255), Color3.fromRGB(190,195,210)
local PURPLE, PURPLE_LT, PURPLE_DK = Color3.fromRGB(157,0,255), Color3.fromRGB(200,80,255), Color3.fromRGB(80,0,140)
local GREEN, RED, YELLOW, BLUE = Color3.fromRGB(0,230,140), Color3.fromRGB(255,90,90), Color3.fromRGB(255,200,60), Color3.fromRGB(80,160,255)
local ORANGE = Color3.fromRGB(255,140,40)
local SWITCH_OFF = Color3.fromRGB(60,60,80)
local SENHA = "2011"
local CORNER_R = 22

local function Corner(p, r) local c = Instance.new("UICorner"); c.CornerRadius = UDim.new(0, r or 12); c.Parent = p; return c end
local function Stroke(p, c, t, tr)
    local s = Instance.new("UIStroke"); s.Color = c or BORDER; s.Thickness = t or 1
    s.Transparency = tr or 0; s.ApplyStrokeMode = Enum.ApplyStrokeMode.Contextual; s.Parent = p; return s
end
local function Gradient(p, c1, c2, rot)
    local g = Instance.new("UIGradient"); g.Color = ColorSequence.new(c1, c2); g.Rotation = rot or 0; g.Parent = p; return g
end
local function Glow(p, color, esc)
    esc = esc or 1
    for _, v in ipairs({{8*esc, 0.94}, {5*esc, 0.88}, {3*esc, 0.78}}) do
        local s = Instance.new("UIStroke"); s.Color = color; s.Thickness = v[1]
        s.Transparency = v[2]; s.ApplyStrokeMode = Enum.ApplyStrokeMode.Border; s.Parent = p
    end
end
local function Lbl(p, txt, sz, pos, font, col)
    local l = Instance.new("TextLabel"); l.BackgroundTransparency = 1; l.Text = txt
    l.TextColor3 = col or TXT; l.TextSize = sz or 13; l.Font = font or Enum.Font.Gotham
    l.Position = pos or UDim2.new(); l.Size = UDim2.new(1,0,0,22)
    l.TextXAlignment = Enum.TextXAlignment.Left; l.Parent = p; return l
end
local function Tw(o, p, d, st, dir)
    TweenService:Create(o, TweenInfo.new(d or 0.2, st or Enum.EasingStyle.Quart, dir or Enum.EasingDirection.Out), p):Play()
end
local function Pulse(o, prop, a, b, t)
    task.spawn(function()
        while o and o.Parent do
            Tw(o, {[prop] = b}, t or 1, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut)
            task.wait(t or 1)
            if not (o and o.Parent) then break end
            Tw(o, {[prop] = a}, t or 1, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut)
            task.wait(t or 1)
        end
    end)
end
local function Shimmer(parent)
    local sh = Instance.new("Frame")
    sh.Size = UDim2.new(0, 60, 1, 0); sh.Position = UDim2.new(-0.2, 0, 0, 0)
    sh.BackgroundColor3 = Color3.new(1,1,1); sh.BackgroundTransparency = 0.85
    sh.BorderSizePixel = 0; sh.ZIndex = 5; sh.Parent = parent; Corner(sh, 30)
    task.spawn(function()
        while sh and sh.Parent do
            sh.Position = UDim2.new(-0.2, 0, 0, 0)
            Tw(sh, {Position = UDim2.new(1.2, 0, 0, 0)}, 1.8, Enum.EasingStyle.Quad)
            task.wait(2.5)
        end
    end)
end
local function Ripple(btn)
    if not btn or not btn.Parent then return end
    local r = Instance.new("Frame")
    r.AnchorPoint = Vector2.new(0.5, 0.5); r.BackgroundColor3 = Color3.new(1,1,1)
    r.BackgroundTransparency = 0.7; r.BorderSizePixel = 0
    r.Size = UDim2.fromOffset(10,10); r.Position = UDim2.fromScale(0.5,0.5)
    r.ZIndex = 5; r.Parent = btn; Corner(r, 100)
    Tw(r, {Size = UDim2.fromOffset(btn.AbsoluteSize.X * 1.8, btn.AbsoluteSize.X * 1.8), BackgroundTransparency = 1}, 0.6, Enum.EasingStyle.Quad)
    task.delay(0.6, function() if r and r.Parent then r:Destroy() end end)
end
local function HookHover(btn, normal, hover, strokeTarget, strokeHover)
    btn.MouseEnter:Connect(function()
        Tw(btn, {BackgroundColor3 = hover or CARD_HOVER}, 0.18)
        if strokeTarget and strokeHover then Tw(strokeTarget, {Color = strokeHover}, 0.18) end
    end)
    btn.MouseLeave:Connect(function()
        Tw(btn, {BackgroundColor3 = normal}, 0.18)
        if strokeTarget and strokeHover then Tw(strokeTarget, {Color = BORDER}, 0.18) end
    end)
end

local Gui = Instance.new("ScreenGui")
Gui.Name = "TropaDoLKK"; Gui.ResetOnSpawn = false
Gui.IgnoreGuiInset = true
Gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
Gui.DisplayOrder = 100
Gui.Parent = PlayerGui

local RGBList, RGBTextColors = {}, {}
local function AddRGB(o) table.insert(RGBList, o) end
local function AddRGBColor(l) table.insert(RGBTextColors, l); return l end

task.spawn(function()
    local t = 0
    while Gui and Gui.Parent do
        t = t + 0.005
        local c = Color3.fromHSV(t % 1, 0.85, 1)
        for _, o in ipairs(RGBList) do if o and o.Parent then pcall(function() o.Color = c end) end end
        for _, o in ipairs(RGBTextColors) do if o and o.Parent then pcall(function() o.TextColor3 = c end) end end
        RunService.RenderStepped:Wait()
    end
end)

-- ===================== ATIVIDADES / NOTIFICAÇÕES =====================
local Atividades, AtividadeFrame = {}, nil
local function AddAtividade(txt, cor)
    cor = cor or TXT_DIM
    table.insert(Atividades, 1, {txt = txt, cor = cor, hora = os.date("%H:%M:%S")})
    if #Atividades > 14 then table.remove(Atividades) end
    if not AtividadeFrame then return end
    for _, c in ipairs(AtividadeFrame:GetChildren()) do if c:IsA("Frame") then c:Destroy() end end
    local y = 0
    for _, a in ipairs(Atividades) do
        local rf = Instance.new("Frame")
        rf.Size = UDim2.new(1, 0, 0, 22); rf.Position = UDim2.fromOffset(0, y)
        rf.BackgroundTransparency = 1; rf.Parent = AtividadeFrame
        local dot = Instance.new("Frame")
        dot.Size = UDim2.fromOffset(6,6); dot.Position = UDim2.fromOffset(0,8)
        dot.BackgroundColor3 = a.cor; dot.BorderSizePixel = 0
        dot.Parent = rf; Corner(dot, 3)
        Lbl(rf, a.txt, 11, UDim2.fromOffset(14,0), Enum.Font.Gotham).Size = UDim2.new(1, -70, 1, 0)
        local hr = Lbl(rf, a.hora, 9, UDim2.new(1,-60,0,0), Enum.Font.Gotham, TXT_DIM)
        hr.Size = UDim2.fromOffset(60, 22); hr.TextXAlignment = Enum.TextXAlignment.Right
        y = y + 22
    end
end

local NotifList = {}
local function Notify(txt, col, icone)
    col = col or PURPLE_LT; icone = icone or "•"
    AddAtividade(txt, col)
    local c = Instance.new("Frame")
    c.Size = UDim2.fromOffset(300, 52)
    c.Position = UDim2.new(1, 360, 0, 14 + #NotifList * 60)
    c.BackgroundColor3 = CARD_BG
    c.ZIndex = 100
    c.Parent = Gui; Corner(c, 14)
    local s = Stroke(c, col, 1); Gradient(c, CARD_BG, BG, 0); AddRGB(s)
    local side = Instance.new("Frame")
    side.Size = UDim2.new(0, 4, 1, -14); side.Position = UDim2.fromOffset(7, 7)
    side.BackgroundColor3 = col; side.BorderSizePixel = 0
    side.ZIndex = 101; side.Parent = c; Corner(side, 2)
    local ico = Instance.new("TextLabel")
    ico.Size = UDim2.fromOffset(28, 28); ico.Position = UDim2.fromOffset(18, 12)
    ico.BackgroundColor3 = col; ico.BackgroundTransparency = 0.82
    ico.Text = icone; ico.TextColor3 = col; ico.TextSize = 16
    ico.Font = Enum.Font.GothamBlack; ico.ZIndex = 101; ico.Parent = c; Corner(ico, 8)
    local l = Lbl(c, txt, 12, UDim2.fromOffset(56, 0), Enum.Font.GothamMedium)
    l.Size = UDim2.new(1, -66, 1, -10); l.TextYAlignment = Enum.TextYAlignment.Center
    l.ZIndex = 101
    local prog = Instance.new("Frame")
    prog.Size = UDim2.new(1, -28, 0, 2); prog.Position = UDim2.new(0, 14, 1, -9)
    prog.BackgroundColor3 = col; prog.BorderSizePixel = 0
    prog.ZIndex = 101; prog.Parent = c; Corner(prog, 1)
    Tw(prog, {Size = UDim2.new(0,0,0,2)}, 3, Enum.EasingStyle.Linear)
    table.insert(NotifList, c)
    Tw(c, {Position = UDim2.new(1, -316, 0, 14 + (#NotifList - 1) * 60)}, 0.35, Enum.EasingStyle.Back)
    task.delay(3, function()
        if c and c.Parent then
            Tw(c, {Position = UDim2.new(1, 360, 0, 14)}, 0.3)
            task.wait(0.35)
            if c and c.Parent then c:Destroy() end
            for i, v in ipairs(NotifList) do if v == c then table.remove(NotifList, i); break end end
            for i, v in ipairs(NotifList) do
                if v and v.Parent then Tw(v, {Position = UDim2.new(1, -316, 0, 14 + (i-1) * 60)}) end
            end
        end
    end)
end

-- ===================== CONFIG =====================
local SAFE_SPEED_LIMIT = 20

local Config = {
    IgnorarTime = true, SempreAtivo = true, BotaoAtivacao = "Esquerdo",
    Predicao = true, DistanciaAimbot = 5000, SemArma = false,
    MostrarFOVCircle = true, FOVCircleSize = 200, FOV_Camera = 70,
    Noclip = false, NoclipKey = "N", OpenKey = "RightShift", FOVAtivo = false,
    StealthMode = true,
}
local Settings = {
    Aimbot = { Enabled=false, Part="Head", FOV=200, Smoothness=1, WallCheck=false },
    Aimlock = { Enabled=false, Velocidade=5, Forca=1.0, Precisao=3, Smoothness=5 },
    Triggerbot = { Enabled=false, Mode="Circle", Radius=150, Delay=0.05 },
    NoRecoil = { Enabled=false, Intensity=0.97 },
    ESP = {
        Enabled=false, Distance=6000, RGB=true,
        ShowName=true, NameMode="Display",
        Box=true, BoxStyle="2D",
        Line=false, Tracers=false,
        Health=true, Weapon=true,
        Skeleton=false, HeadDot=false,
        ShowBackpack=false, Tool=true,
        ShowTeam=true, TeamCheck=false, ColorByTeam=false,
        ESPColor = Color3.fromRGB(0,230,140),
        ChamsColor = Color3.fromRGB(0,230,140),
        VisibleColor = Color3.fromRGB(255,200,60),
        WallColor = Color3.fromRGB(255,80,80),
    },
    Chams = { Enabled=false, RGB=true, ChamsColor=Color3.fromRGB(0,230,140) },
    GodMode = { Enabled=false }, AutoRevive = { Enabled=false },
    WalkSpeed = { Enabled=false, Value=SAFE_SPEED_LIMIT },
    Underground = { Enabled=false, Depth=1 },
    Spinbot = { Enabled=false, Speed=360 },
    AutoJump = { Enabled=false }, Colar = { Enabled=false, Target=nil },
    GiantGun = { Enabled=false }, Teams = {},
    World = {
        TimeEnabled = false, TimeValue = 17,
        BrightnessEnabled = false, BrightnessValue = 2,
        RemoveFog = false, Fullbright = false,
    },
    Performance = {
        FPSLimit = 970, ShowFPS = false,
        RemoveTextures = false, AntiLag = false,
        RemoveShadows = false, AntiAFK = false,
    },
    Vehicle = {
        ESPEnabled = false, ESPDistance = 2500,
        ESPColor = Color3.fromRGB(0,230,140),
        ChamsColor = Color3.fromRGB(0,230,140),
        VisibleColor = Color3.fromRGB(255,200,60),
        NoClip = false, InfiniteFuel = false, Invincible = false,
        TireProtection = false, AntiPIT = false, EngineBoost = false,
        Horsepower = 10000,
        TrunkItems = nil,
        TrunkAutoStore = false, TrunkHealthThreshold = 5,
        Fly = false, FlySpeed = 600, FlyBoostMult = 4.25, FlyAccel = 1,
        Fling = false, SelectedVehicle = nil,
        EnterOnKey = false,
    },
    Config = { Theme = "Padrão", MenuKey = "RightShift", SelectedJob = "PM" },
}
local Amigos = {}
local Arma = false
local EstadoCanCollide = {}
local NoclipTimer, UndergroundTimer, ColarTimer, GiantGunTimer = 0, 0, 0, 0

local S = {
    Panel=nil, Login=nil, LoginCard=nil, InputWrap=nil,
    PassInput=nil, EnterBtn=nil, ErrorLbl=nil,
    Sidebar=nil, Content=nil, SidebarAberta=true, Header=nil,
    SearchBox=nil, pV3=nil, FPSLabel=nil, VehicleListLabel=nil, TrunkItemsLabel=nil,
    TabBtns={}, Pages={},
    Tabs = {
        {name="Principal", icon="🏠", desc="Dashboard"},
        {name="Combate", icon="⚔", desc="Aimbot & Trigger"},
        {name="Visual", icon="👁", desc="ESP & Chams"},
        {name="Movimento", icon="🏃", desc="Speed & Fly"},
        {name="Jogador", icon="👤", desc="Amigos"},
        {name="Util", icon="🛠", desc="Ferramentas"},
        {name="Mundo", icon="🌍", desc="Iluminação"},
        {name="Desempenho", icon="⚡", desc="FPS & Otimização"},
        {name="Veículos", icon="🚗", desc="Vehicle Toolkit"},
        {name="Config", icon="⚙", desc="Ajustes"},
    },
}

-- ===================== HELPERS DE JOGO =====================
local function GetHumanoid()
    local c = LocalPlayer.Character
    return c and c:FindFirstChildOfClass("Humanoid")
end
local function GetHRP()
    local c = LocalPlayer.Character
    return c and c:FindFirstChild("HumanoidRootPart")
end

local function CheckArma()
    local c = LocalPlayer.Character
    if not c then Arma = false; return end
    for _, i in pairs(c:GetChildren()) do if i:IsA("Tool") then Arma = true; return end end
    Arma = false
end
task.spawn(function() while true do task.wait(0.15); pcall(CheckArma) end end)

local function EhInimigo(j)
    if not j then return false end
    if Config.IgnorarTime then return true end
    if not j.Team or not LocalPlayer.Team then return true end
    return j.Team ~= LocalPlayer.Team
end
local function EhAmigo(j) return Amigos[j.Name] == true end
local function AddAmigo(n) Amigos[n] = true; Notify("Adicionado: "..n, GREEN, "+") end
local function RemAmigo(n) Amigos[n] = nil; Notify("Removido: "..n, RED, "−") end
local function LimparAmigos() Amigos = {}; Notify("Lista limpa", RED, "!") end

local function AplicarNoclip(estado)
    local c = LocalPlayer.Character
    if not c then return end
    if estado then
        EstadoCanCollide = {}
        for _, p in pairs(c:GetDescendants()) do
            if p:IsA("BasePart") then
                EstadoCanCollide[p] = p.CanCollide
                p.CanCollide = false
            end
        end
        NoclipTimer = os.clock() + AutoDisableTimers.noclip
    else
        for p, v in pairs(EstadoCanCollide) do
            if p and p.Parent then pcall(function() p.CanCollide = v end) end
        end
        EstadoCanCollide = {}
        NoclipTimer = 0
    end
end

-- ===================== SPEED HUB =====================
local BASE_SPEED = 16
do
    local h = GetHumanoid()
    if h then BASE_SPEED = h.WalkSpeed end
end
local CUR_SPEED = LocalPlayer:GetAttribute("SpeedHubValue") or BASE_SPEED
if CUR_SPEED > SAFE_SPEED_LIMIT then CUR_SPEED = SAFE_SPEED_LIMIT end
local SpeedValueBox, SpeedAtivo = nil, false

local function SetSpeed(v)
    v = math.clamp(math.floor(v + 0.5), 0, 200)
    if Config.StealthMode and v > SAFE_SPEED_LIMIT then
        v = SAFE_SPEED_LIMIT
        Notify("⚠️ Speed limitado a "..SAFE_SPEED_LIMIT.." (anti-kick)", ORANGE, "🛡")
    end
    CUR_SPEED = v
    pcall(function() LocalPlayer:SetAttribute("SpeedHubValue", v) end)
    if SpeedValueBox then SpeedValueBox.Text = tostring(v) end
    if S and S.pV3 then S.pV3.Text = tostring(v) end
    if SpeedAtivo then
        local h = GetHumanoid()
        if h then h.WalkSpeed = v end
    end
end

LocalPlayer.CharacterAdded:Connect(function(ch)
    task.wait(0.5)
    local h = ch:FindFirstChildOfClass("Humanoid")
    if h then BASE_SPEED = h.WalkSpeed end
    -- BUG FIX #12: reset correto do restore stack no respawn
    RestoreStack["__ws_orig"] = nil
    EstadoCanCollide = {}
    NoclipTimer = 0
    if Config.Noclip then AplicarNoclip(true) end
    if SpeedAtivo and h then h.WalkSpeed = CUR_SPEED end
end)

RunService.Heartbeat:Connect(function()
    local h = GetHumanoid()
    if not h then return end
    if SpeedAtivo and not Settings.WalkSpeed.Enabled then
        if h.WalkSpeed ~= CUR_SPEED then h.WalkSpeed = CUR_SPEED end
    end
end)

-- ===================== WATCHDOG =====================
task.spawn(function()
    while Gui and Gui.Parent do
        task.wait(5)
        if Config.Noclip and NoclipTimer > 0 and os.clock() > NoclipTimer then
            Config.Noclip = false
            AplicarNoclip(false)
            Notify("⚠️ Noclip auto-desligado (30s)", ORANGE, "🛡")
        end
        if Settings.Underground.Enabled and UndergroundTimer > 0 and os.clock() > UndergroundTimer then
            Settings.Underground.Enabled = false
            local r = GetHRP()
            if r and r:FindFirstChild("UG") then r.UG:Destroy() end
            Notify("⚠️ Underground auto-desligado (15s)", ORANGE, "🛡")
            UndergroundTimer = 0
        end
        if Settings.Colar.Enabled and ColarTimer > 0 and os.clock() > ColarTimer then
            Settings.Colar.Enabled = false
            Notify("⚠️ Colar auto-desligado (20s)", ORANGE, "🛡")
            ColarTimer = 0
        end
        if Settings.GiantGun.Enabled and GiantGunTimer > 0 and os.clock() > GiantGunTimer then
            Settings.GiantGun.Enabled = false
            Notify("⚠️ GiantGun auto-desligado (60s)", ORANGE, "🛡")
            GiantGunTimer = 0
        end
        if Config.StealthMode and SpeedAtivo and CUR_SPEED > SAFE_SPEED_LIMIT then
            SetSpeed(SAFE_SPEED_LIMIT)
        end
    end
end)

local Drawing_Lib = nil
do local ok, r = pcall(function() return Drawing end); if ok and r then Drawing_Lib = r end end

local function IsVisible(t)
    if not t or not t.Character then return false end
    local p = t.Character:FindFirstChild("Head"); if not p then return false end
    local rp = RaycastParams.new()
    rp.FilterType = Enum.RaycastFilterType.Exclude
    local ig = {LocalPlayer.Character, t.Character}
    local h = t.Character:FindFirstChildOfClass("Humanoid")
    if h and h.SeatPart then
        local v = h.SeatPart.Parent
        if v then
            table.insert(ig, v)
            for _, c in ipairs(v:GetDescendants()) do table.insert(ig, c) end
        end
    end
    rp.FilterDescendantsInstances = ig
    return Workspace:Raycast(Camera.CFrame.Position, (p.Position - Camera.CFrame.Position).Unit * 500, rp) == nil
end

-- BUG FIX #11: Neck é Motor6D em R6 — usar Torso se não houver Neck Part
local function GetPart(ch)
    if Settings.Aimbot.Part == "Head" then
        return ch:FindFirstChild("Head")
    elseif Settings.Aimbot.Part == "Neck" then
        local neck = ch:FindFirstChild("Neck")
        if neck and neck:IsA("BasePart") then return neck end
        return ch:FindFirstChild("UpperTorso") or ch:FindFirstChild("Torso")
    elseif Settings.Aimbot.Part == "Chest" then
        return ch:FindFirstChild("UpperTorso") or ch:FindFirstChild("Torso")
    end
    return ch:FindFirstChild("Head")
end

local function PredictPart(part)
    if not part or not part.Parent then return nil end
    if not Config.Predicao then return { Position = part.Position } end
    local vel = part.AssemblyLinearVelocity
    if not vel or vel.Magnitude < 1 then return { Position = part.Position } end
    return { Position = part.Position + vel * 0.1 }
end

local function GetTarget()
    local best, bestD = nil, Settings.Aimbot.FOV
    local c = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
    local myR = GetHRP()
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= LocalPlayer and p.Character and not EhAmigo(p) and EhInimigo(p) then
            local h = p.Character:FindFirstChildOfClass("Humanoid")
            if h and h.Health > 0 then
                local pt = GetPart(p.Character)
                if pt then
                    if myR and Config.DistanciaAimbot > 0 then
                        if (pt.Position - myR.Position).Magnitude > Config.DistanciaAimbot then continue end
                    end
                    local pred = PredictPart(pt)
                    if pred then
                        local sp = Camera:WorldToViewportPoint(pred.Position)
                        if sp.Z > 0 then
                            if Settings.Aimbot.WallCheck and not IsVisible(p) then continue end
                            local d = (Vector2.new(sp.X, sp.Y) - c).Magnitude
                            if d < bestD then bestD = d; best = {J = p, Part = pt} end
                        end
                    end
                end
            end
        end
    end
    return best
end

local FOVCircleDrawing = nil
local function GarantirFOVCircle()
    if FOVCircleDrawing or not Drawing_Lib then return end
    local ok, result = pcall(function()
        local circle = Drawing_Lib.new("Circle")
        circle.Thickness = 2
        circle.Color = PURPLE
        circle.Filled = false
        circle.NumSides = 80
        circle.Transparency = 1
        circle.Visible = false
        return circle
    end)
    if ok then FOVCircleDrawing = result end
end

task.spawn(function()
    while Gui and Gui.Parent do
        if Config.MostrarFOVCircle and (Settings.Aimbot.Enabled or Settings.Aimlock.Enabled) then
            GarantirFOVCircle()
        end
        if FOVCircleDrawing then
            if Config.MostrarFOVCircle and (Settings.Aimbot.Enabled or Settings.Aimlock.Enabled) then
                FOVCircleDrawing.Visible = true
                FOVCircleDrawing.Radius = Settings.Aimbot.FOV
                FOVCircleDrawing.Position = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
                FOVCircleDrawing.Color = Settings.ESP.RGB and Color3.fromHSV(tick() % 1, 0.85, 1) or PURPLE
            else
                FOVCircleDrawing.Visible = false
            end
        end
        task.wait(0.05)
    end
end)

-- ===================== LOOPS DE COMBATE =====================
local ALVO = nil
local function AimbotLoop()
    if not (Settings.Aimbot.Enabled or Settings.Aimlock.Enabled) then ALVO = nil; return end
    local m1 = UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton1)
    local m2 = UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2)
    local ok = false
    if Config.SempreAtivo then
        if Config.BotaoAtivacao == "Esquerdo" then ok = m1 else ok = m2 end
    else ok = m1 or m2 end
    if not ok then ALVO = nil; return end
    if Config.SemArma and not Arma then ALVO = nil; return end
    local t = nil
    if ALVO then
        local plr = ALVO.J
        if plr and plr.Parent and plr.Character then
            local h = plr.Character:FindFirstChildOfClass("Humanoid")
            if h and h.Health > 0 and not EhAmigo(plr) and EhInimigo(plr) then
                local pt = GetPart(plr.Character)
                if pt then
                    local pred = PredictPart(pt)
                    if pred then
                        local sp = Camera:WorldToViewportPoint(pred.Position)
                        if sp.Z > 0 then
                            local c = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
                            local d = (Vector2.new(sp.X, sp.Y) - c).Magnitude
                            if d <= Settings.Aimbot.FOV then
                                if not Settings.Aimbot.WallCheck or IsVisible(plr) then
                                    t = {J = plr, Part = pt}
                                end
                            end
                        end
                    end
                end
            end
        end
        if not t then ALVO = nil end
    end
    if not t then t = GetTarget(); if t then ALVO = t end end
    if not t or not t.Part then return end
    local pred = PredictPart(t.Part)
    if not pred then return end
    local look = CFrame.new(Camera.CFrame.Position, pred.Position)
    if Settings.Aimlock.Enabled then
        local v = math.max(Settings.Aimlock.Velocidade, 1)
        local f = Settings.Aimlock.Forca
        local p = math.max(Settings.Aimlock.Precisao, 1)
        Camera.CFrame = Camera.CFrame:Lerp(look, math.clamp((1/v) * f, 0.01, 1))
        if p > 1 then Camera.CFrame = Camera.CFrame:Lerp(look, 1/p) end
    else
        local s = math.max(Settings.Aimbot.Smoothness, 1)
        Camera.CFrame = (s <= 1) and look or Camera.CFrame:Lerp(look, 1/s)
    end
end

local function TriggerLoop()
    if not Settings.Triggerbot.Enabled then return end
    local c = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("Head") and EhInimigo(p) and not EhAmigo(p) then
            local sp = Camera:WorldToViewportPoint(p.Character.Head.Position)
            if sp.Z > 0 and (Vector2.new(sp.X, sp.Y) - c).Magnitude <= Settings.Triggerbot.Radius then
                FireMouse1Press()
                task.wait(Settings.Triggerbot.Delay)
                FireMouse1Release()
                break
            end
        end
    end
end

-- BUG FIX #16: NoRecoil "sensibilidade" — não é recoil real, mas mantém comportamento local
local function NoRecoilLoop()
    if not Settings.NoRecoil.Enabled then
        if UserInputService.MouseDeltaSensitivity < 1 then
            pcall(function() UserInputService.MouseDeltaSensitivity = 1 end)
        end
        return
    end
    pcall(function()
        UserInputService.MouseDeltaSensitivity = math.max(0.1, 1 - Settings.NoRecoil.Intensity * 0.5)
    end)
end

local function GodLoop()
    if not Settings.GodMode.Enabled then return end
    local c = LocalPlayer.Character; if not c then return end
    local h = c:FindFirstChildOfClass("Humanoid"); if not h then return end
    pcall(function() if h.Health < h.MaxHealth then h.Health = h.MaxHealth end end)
    pcall(function() h:SetStateEnabled(Enum.HumanoidStateType.Dead, false) end)
end

-- BUG FIX #7: ReviveLoop removido do envio de remotes sem args (dossiê: servidor push-only)
local function ReviveLoop()
    if not Settings.AutoRevive.Enabled then return end
    local c = LocalPlayer.Character; if not c then return end
    local h = c:FindFirstChildOfClass("Humanoid")
    if h and h.Health <= 0 then
        pcall(function() h.Health = h.MaxHealth end)
    end
end

local lastT = tick()
local function MoveLoop()
    local c = LocalPlayer.Character; if not c then return end
    local r = c:FindFirstChild("HumanoidRootPart")
    local h = c:FindFirstChildOfClass("Humanoid")

    if Settings.WalkSpeed.Enabled and r and h then
        if h.WalkSpeed ~= 0 and RestoreStack["__ws_orig"] == nil then
            RestoreStack["__ws_orig"] = h.WalkSpeed
        end
        h.WalkSpeed = 0
        local mv = h.MoveDirection
        if mv.Magnitude > 0 then
            r.CFrame = r.CFrame + mv * Settings.WalkSpeed.Value * (tick() - lastT)
        end
    elseif h and RestoreStack["__ws_orig"] ~= nil then
        pcall(function() h.WalkSpeed = RestoreStack["__ws_orig"] end)
        RestoreStack["__ws_orig"] = nil
    end

    if Settings.Underground.Enabled and r then
        if UndergroundTimer == 0 then UndergroundTimer = os.clock() + AutoDisableTimers.underground end
        if not r:FindFirstChild("UG") then
            local bp = Instance.new("BodyPosition", r)
            bp.Name = "UG"
            bp.MaxForce = Vector3.new(0, 5e4, 0)
            bp.Position = r.Position - Vector3.new(0, Settings.Underground.Depth, 0)
        else
            r.UG.Position = r.Position - Vector3.new(0, Settings.Underground.Depth, 0)
        end
    else
        if r and r:FindFirstChild("UG") then r.UG:Destroy() end
        UndergroundTimer = 0
    end

    if Settings.Spinbot.Enabled and r then
        r.CFrame = r.CFrame * CFrame.fromEulerAnglesXYZ(0, math.rad(Settings.Spinbot.Speed/60), 0)
    end
    if Settings.AutoJump.Enabled and h and h.FloorMaterial ~= Enum.Material.Air then
        h.Jump = true
    end

    if Settings.GiantGun.Enabled then
        if GiantGunTimer == 0 then GiantGunTimer = os.clock() + AutoDisableTimers.giantgun end
        for _, t in ipairs(c:GetChildren()) do
            if t:IsA("Tool") and t:FindFirstChild("Handle") then
                if RestoreStack[t] == nil then RestoreStack[t] = t.Handle.Size end
                pcall(function() t.Handle.Size = Vector3.new(3,3,3) end)
            end
        end
    else
        GiantGunTimer = 0
        for _, t in ipairs(c:GetChildren()) do
            if t:IsA("Tool") and t:FindFirstChild("Handle") and RestoreStack[t] ~= nil then
                pcall(function() t.Handle.Size = RestoreStack[t] end)
                RestoreStack[t] = nil
            end
        end
    end
    lastT = tick()
end

local function ColarLoop()
    if not Settings.Colar.Enabled or not Settings.Colar.Target then return end
    if ColarTimer == 0 then ColarTimer = os.clock() + AutoDisableTimers.colar end
    local mr = GetHRP()
    local tr = Settings.Colar.Target.Character and Settings.Colar.Target.Character:FindFirstChild("HumanoidRootPart")
    if mr and tr then mr.CFrame = tr.CFrame * CFrame.new(0, 0, -3) end
end

-- ===================== ESP / CHAMS =====================
local ESPCache, ChamsCache, HeadDotCache, SkeletonCache = {}, {}, {}, {}
local hue = 0
local function HSV(h, s, v) return Color3.fromHSV(h % 1, s or 0.8, v or 1) end
local function CleanupESP(p)
    if ESPCache[p] then
        for _, o in pairs(ESPCache[p]) do pcall(function() o:Remove() end) end
        ESPCache[p] = nil
    end
end
local function CleanupAllESP() for p in pairs(ESPCache) do CleanupESP(p) end end
local function CleanupHeadDots()
    for _, hl in pairs(HeadDotCache) do pcall(function() hl:Destroy() end) end
    HeadDotCache = {}
    for _, p in ipairs(Players:GetPlayers()) do
        if p.Character then
            local head = p.Character:FindFirstChild("Head")
            if head then
                for _, c in ipairs(head:GetChildren()) do
                    if c:IsA("BillboardGui") and c.Name == "LKK_HeadDot" then
                        pcall(function() c:Destroy() end)
                    end
                end
            end
        end
    end
end
local function CleanupSkeletons()
    for _, tbl in pairs(SkeletonCache) do
        for _, ln in pairs(tbl) do pcall(function() ln:Remove() end) end
    end
    SkeletonCache = {}
end

local function ESPLoop()
    hue = (hue + 0.005) % 1
    if not Drawing_Lib then return end
    if not Settings.ESP.Enabled then
        CleanupAllESP(); CleanupHeadDots(); CleanupSkeletons(); return
    end
    local baseCol = Settings.ESP.RGB and HSV(hue, 0.85, 1) or Settings.ESP.ESPColor
    local mr = GetHRP()
    if not mr then return end
    for _, p in ipairs(Players:GetPlayers()) do
        local ok, h, r, hum, d = false, nil, nil, nil, nil
        if p ~= LocalPlayer and not EhAmigo(p) and EhInimigo(p) and p.Character then
            if Settings.ESP.TeamCheck and LocalPlayer.Team and p.Team and p.Team == LocalPlayer.Team then
            else
                h = p.Character:FindFirstChild("Head")
                r = p.Character:FindFirstChild("HumanoidRootPart")
                hum = p.Character:FindFirstChildOfClass("Humanoid")
                if h and r and hum and hum.Health > 0 then
                    d = (r.Position - mr.Position).Magnitude
                    if d <= Settings.ESP.Distance then ok = true end
                end
            end
        end
        if ok then
            local col = baseCol
            if Settings.ESP.ColorByTeam and p.Team then
                col = (p.Team == LocalPlayer.Team) and GREEN or RED
            elseif Settings.ESP.RGB then
                col = baseCol
            elseif IsVisible(p) then
                col = Settings.ESP.VisibleColor
            else
                col = Settings.ESP.WallColor
            end
            if not ESPCache[p] then
                ESPCache[p] = {
                    box = Drawing_Lib.new("Square"),
                    boxCorner = Drawing_Lib.new("Square"),
                    line = Drawing_Lib.new("Line"),
                    tracer = Drawing_Lib.new("Line"),
                    name = Drawing_Lib.new("Text"),
                    dist = Drawing_Lib.new("Text"),
                    hp = Drawing_Lib.new("Square"),
                    wp = Drawing_Lib.new("Text"),
                    tm = Drawing_Lib.new("Text"),
                    bp = Drawing_Lib.new("Text"),
                }
                ESPCache[p].box.Filled = false; ESPCache[p].box.Thickness = 2
                ESPCache[p].boxCorner.Filled = false; ESPCache[p].boxCorner.Thickness = 2
                ESPCache[p].hp.Filled = true
            end
            local e = ESPCache[p]
            local hp1 = Camera:WorldToViewportPoint(h.Position + Vector3.new(0, 0.5, 0))
            local hp2 = Camera:WorldToViewportPoint(r.Position - Vector3.new(0, 3, 0))
            if hp1.Z > 0 and hp2.Z > 0 then
                local height = math.abs(hp2.Y - hp1.Y)
                local width = height * 0.6
                if Settings.ESP.Box then
                    e.box.Visible = false; e.boxCorner.Visible = false
                    if Settings.ESP.BoxStyle == "2D" then
                        e.box.Position = Vector2.new(hp1.X - width/2, hp1.Y)
                        e.box.Size = Vector2.new(width, height)
                        e.box.Color = col; e.box.Visible = true
                    elseif Settings.ESP.BoxStyle == "3D" then
                        e.box.Position = Vector2.new(hp1.X - width/2, hp1.Y)
                        e.box.Size = Vector2.new(width, height)
                        e.box.Color = col; e.box.Thickness = 3; e.box.Visible = true
                    elseif Settings.ESP.BoxStyle == "Corner" then
                        e.boxCorner.Position = Vector2.new(hp1.X - width/2, hp1.Y)
                        e.boxCorner.Size = Vector2.new(width, height)
                        e.boxCorner.Color = col; e.boxCorner.Thickness = 2; e.boxCorner.Visible = true
                    end
                else e.box.Visible = false; e.boxCorner.Visible = false end
                if Settings.ESP.Line then
                    e.line.From = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y)
                    e.line.To = Vector2.new(hp1.X, hp2.Y)
                    e.line.Color = col; e.line.Visible = true
                else e.line.Visible = false end
                if Settings.ESP.Tracers then
                    e.tracer.From = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y - 60)
                    e.tracer.To = Vector2.new(hp1.X, hp1.Y)
                    e.tracer.Color = col; e.tracer.Visible = true
                else e.tracer.Visible = false end
                if Settings.ESP.ShowName then
                    e.name.Text = (Settings.ESP.NameMode == "UserName") and p.Name or ("@"..p.Name)
                    e.name.Position = Vector2.new(hp1.X, hp1.Y - 32)
                    e.name.Color = col; e.name.Size = 13
                    e.name.Center = true; e.name.Outline = true; e.name.Visible = true
                else e.name.Visible = false end
                if Settings.ESP.ShowTeam and Settings.Teams[p.Name] then
                    e.tm.Text = "["..Settings.Teams[p.Name].."]"
                    e.tm.Position = Vector2.new(hp1.X, hp1.Y - 48)
                    e.tm.Color = TXT; e.tm.Size = 10
                    e.tm.Center = true; e.tm.Outline = true; e.tm.Visible = true
                else e.tm.Visible = false end
                e.dist.Text = math.floor(d).."m · "..math.floor(hum.Health).."hp"
                e.dist.Position = Vector2.new(hp1.X, hp1.Y - 18)
                e.dist.Color = col; e.dist.Size = 11
                e.dist.Center = true; e.dist.Outline = true; e.dist.Visible = true
                if Settings.ESP.Health then
                    local pct = hum.Health / hum.MaxHealth
                    e.hp.Position = Vector2.new(hp1.X - width/2 - 5, hp1.Y + height * (1 - pct))
                    e.hp.Size = Vector2.new(3, height * pct)
                    e.hp.Color = pct > 0.5 and Color3.fromRGB(0,255,0) or (pct > 0.25 and Color3.fromRGB(255,255,0) or Color3.fromRGB(255,0,0))
                    e.hp.Visible = true
                else e.hp.Visible = false end
                if Settings.ESP.Weapon then
                    e.wp.Text = "?"
                    for _, t in ipairs(p.Character:GetChildren()) do
                        if t:IsA("Tool") then e.wp.Text = t.Name; break end
                    end
                    e.wp.Position = Vector2.new(hp1.X, hp2.Y + 16)
                    e.wp.Color = col; e.wp.Size = 11
                    e.wp.Center = true; e.wp.Outline = true; e.wp.Visible = true
                else e.wp.Visible = false end
                if Settings.ESP.ShowBackpack then
                    local cnt = 0
                    local bkp = p:FindFirstChild("Backpack")
                    if bkp then
                        for _, item in ipairs(bkp:GetChildren()) do if item:IsA("Tool") then cnt = cnt + 1 end end
                    end
                    e.bp.Text = "🎒 "..cnt
                    e.bp.Position = Vector2.new(hp1.X, hp2.Y + 32)
                    e.bp.Color = col; e.bp.Size = 11
                    e.bp.Center = true; e.bp.Outline = true; e.bp.Visible = true
                else e.bp.Visible = false end
            else
                for _, o in pairs(e) do o.Visible = false end
            end
        else
            CleanupESP(p)
        end
    end

    if Settings.ESP.Skeleton then
        for _, p in ipairs(Players:GetPlayers()) do
            if p ~= LocalPlayer and p.Character and EhInimigo(p) and not EhAmigo(p) then
                local ch = p.Character
                local hum = ch:FindFirstChildOfClass("Humanoid")
                if hum and hum.Health > 0 then
                    if not SkeletonCache[p] or SkeletonCache[p]._char ~= ch then
                        if SkeletonCache[p] then
                            for _, ln in pairs(SkeletonCache[p]) do pcall(function() ln:Remove() end) end
                        end
                        SkeletonCache[p] = {
                            _char = ch,
                            head = Drawing_Lib.new("Line"), torso = Drawing_Lib.new("Line"),
                            larm = Drawing_Lib.new("Line"), rarm = Drawing_Lib.new("Line"),
                            lleg = Drawing_Lib.new("Line"), rleg = Drawing_Lib.new("Line"),
                        }
                        for k, ln in pairs(SkeletonCache[p]) do
                            if k ~= "_char" then ln.Thickness = 1; ln.Color = Settings.ESP.ESPColor end
                        end
                    end
                    local sk = SkeletonCache[p]
                    local function wp(part)
                        local pa = ch:FindFirstChild(part)
                        if not pa or not pa:IsA("BasePart") then return nil end
                        local sp = Camera:WorldToViewportPoint(pa.Position)
                        if sp.Z <= 0 then return nil end
                        return Vector2.new(sp.X, sp.Y)
                    end
                    local Head = wp("Head"); local UT = wp("UpperTorso") or wp("Torso")
                    local LT = wp("LowerTorso") or wp("Torso")
                    local LH = wp("LeftHand") or wp("Left Arm"); local RH = wp("RightHand") or wp("Right Arm")
                    local LF = wp("LeftFoot") or wp("Left Leg"); local RF = wp("RightFoot") or wp("Right Leg")
                    if Head and UT then sk.head.From = Head; sk.head.To = UT; sk.head.Visible = true else sk.head.Visible = false end
                    if UT and LT then sk.torso.From = UT; sk.torso.To = LT; sk.torso.Visible = true else sk.torso.Visible = false end
                    if UT and LH then sk.larm.From = UT; sk.larm.To = LH; sk.larm.Visible = true else sk.larm.Visible = false end
                    if UT and RH then sk.rarm.From = UT; sk.rarm.To = RH; sk.rarm.Visible = true else sk.rarm.Visible = false end
                    if LT and LF then sk.lleg.From = LT; sk.lleg.To = LF; sk.lleg.Visible = true else sk.lleg.Visible = false end
                    if LT and RF then sk.rleg.From = LT; sk.rleg.To = RF; sk.rleg.Visible = true else sk.rleg.Visible = false end
                end
            end
        end
    else
        CleanupSkeletons()
    end

    if Settings.ESP.HeadDot then
        for _, p in ipairs(Players:GetPlayers()) do
            if p ~= LocalPlayer and p.Character and EhInimigo(p) and not EhAmigo(p) then
                local head = p.Character:FindFirstChild("Head")
                if head then
                    if not HeadDotCache[p] or HeadDotCache[p].Parent ~= head then
                        if HeadDotCache[p] then HeadDotCache[p]:Destroy() end
                        local bb = Instance.new("BillboardGui")
                        bb.Name = "LKK_HeadDot"
                        bb.Size = UDim2.fromOffset(14, 14)
                        bb.AlwaysOnTop = true
                        bb.Adornee = head
                        bb.Parent = head
                        local dot = Instance.new("Frame", bb)
                        dot.Size = UDim2.fromScale(1,1)
                        dot.BackgroundColor3 = Settings.ESP.ESPColor
                        dot.BorderSizePixel = 0
                        Instance.new("UICorner", dot).CornerRadius = UDim.new(1,0)
                        local st = Instance.new("UIStroke", dot)
                        st.Color = Color3.new(0,0,0); st.Thickness = 1
                        HeadDotCache[p] = bb
                    end
                    HeadDotCache[p].Enabled = true
                    local dot = HeadDotCache[p]:FindFirstChildOfClass("Frame")
                    if dot then dot.BackgroundColor3 = Settings.ESP.RGB and HSV(hue, 0.85, 1) or Settings.ESP.ESPColor end
                end
            end
        end
    else
        CleanupHeadDots()
    end
end

local function UpdateChams()
    if not Settings.Chams.Enabled then
        for _, hl in pairs(ChamsCache) do pcall(function() hl:Destroy() end) end
        ChamsCache = {}
        return
    end
    hue = (hue + 0.01) % 1
    local col = Settings.Chams.RGB and HSV(hue, 1, 1) or Settings.Chams.ChamsColor
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= LocalPlayer and not EhAmigo(p) and EhInimigo(p) and p.Character then
            local hum = p.Character:FindFirstChildOfClass("Humanoid")
            if hum and hum.Health > 0 then
                if not ChamsCache[p] or ChamsCache[p].Parent ~= p.Character then
                    if ChamsCache[p] then pcall(function() ChamsCache[p]:Destroy() end) end
                    local hl = Instance.new("Highlight")
                    hl.Adornee = p.Character
                    hl.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
                    hl.FillColor = col; hl.FillTransparency = 0.4
                    hl.OutlineColor = col; hl.OutlineTransparency = 0
                    hl.Parent = p.Character
                    ChamsCache[p] = hl
                else
                    ChamsCache[p].FillColor = col
                    ChamsCache[p].OutlineColor = col
                end
            end
        end
    end
end

-- ===================== MUNDO =====================
-- BUG FIX #9: restaura MoonAngularSize também
local function AplicarMundo()
    if Settings.World.TimeEnabled then
        SaveOrig("__Light_ClockTime", Lighting.ClockTime)
        pcall(function() Lighting.ClockTime = Settings.World.TimeValue end)
    elseif RestoreStack["__Light_ClockTime"] ~= nil then
        pcall(function() Lighting.ClockTime = RestoreStack["__Light_ClockTime"] end)
        RestoreStack["__Light_ClockTime"] = nil
    end
    if Settings.World.BrightnessEnabled then
        SaveOrig("__Light_Brightness", Lighting.Brightness)
        pcall(function() Lighting.Brightness = Settings.World.BrightnessValue end)
    elseif RestoreStack["__Light_Brightness"] ~= nil then
        pcall(function() Lighting.Brightness = RestoreStack["__Light_Brightness"] end)
        RestoreStack["__Light_Brightness"] = nil
    end
    if Settings.World.RemoveFog then
        SaveOrig("__Light_FogEnd", Lighting.FogEnd)
        SaveOrig("__Light_FogStart", Lighting.FogStart)
        pcall(function()
            Lighting.FogEnd = 100000; Lighting.FogStart = 100000
            for _, c in ipairs(Lighting:GetChildren()) do
                if c:IsA("Atmosphere") then SaveOrig(c, c.Density); c.Density = 0 end
            end
        end)
    else
        if RestoreStack["__Light_FogEnd"] ~= nil then
            pcall(function() Lighting.FogEnd = RestoreStack["__Light_FogEnd"] end); RestoreStack["__Light_FogEnd"] = nil
        end
        if RestoreStack["__Light_FogStart"] ~= nil then
            pcall(function() Lighting.FogStart = RestoreStack["__Light_FogStart"] end); RestoreStack["__Light_FogStart"] = nil
        end
        for _, c in ipairs(Lighting:GetChildren()) do
            if c:IsA("Atmosphere") and RestoreStack[c] ~= nil then
                pcall(function() c.Density = RestoreStack[c] end); RestoreStack[c] = nil
            end
        end
    end
    if Settings.World.Fullbright then
        SaveOrig("__Light_Ambient", Lighting.Ambient)
        SaveOrig("__Light_Outdoor", Lighting.OutdoorAmbient)
        pcall(function()
            Lighting.Ambient = Color3.fromRGB(255,255,255)
            Lighting.OutdoorAmbient = Color3.fromRGB(255,255,255)
            Lighting.Brightness = 5; Lighting.ClockTime = 12
            for _, c in ipairs(Lighting:GetChildren()) do
                if c:IsA("Atmosphere") then SaveOrig(c, c.Density); c.Density = 0 end
                if c:IsA("Sky") then
                    SaveOrig(c, c.SunAngularSize)
                    SaveOrig(c, c.MoonAngularSize)  -- FIX
                    c.SunAngularSize = 0; c.MoonAngularSize = 0
                end
            end
        end)
    else
        if RestoreStack["__Light_Ambient"] ~= nil then
            pcall(function() Lighting.Ambient = RestoreStack["__Light_Ambient"] end); RestoreStack["__Light_Ambient"] = nil
        end
        if RestoreStack["__Light_Outdoor"] ~= nil then
            pcall(function() Lighting.OutdoorAmbient = RestoreStack["__Light_Outdoor"] end); RestoreStack["__Light_Outdoor"] = nil
        end
        for _, c in ipairs(Lighting:GetChildren()) do
            if c:IsA("Sky") and RestoreStack[c] ~= nil then
                pcall(function()
                    c.SunAngularSize = RestoreStack[c].SunAngularSize or c.SunAngularSize
                    c.MoonAngularSize = RestoreStack[c].MoonAngularSize or c.MoonAngularSize
                end)
                RestoreStack[c] = nil
            end
        end
    end
end
task.spawn(function()
    while Gui and Gui.Parent do
        pcall(AplicarMundo)
        task.wait(0.5)
    end
end)

-- ===================== DESEMPENHO =====================
local fpsCount, fpsTimer, fpsValue = 0, tick(), 0
RunService.RenderStepped:Connect(function()
    fpsCount = fpsCount + 1
    if tick() - fpsTimer >= 1 then
        fpsValue = fpsCount; fpsCount = 0; fpsTimer = tick()
        if S.FPSLabel then S.FPSLabel.Text = fpsValue.." FPS" end
    end
end)

task.spawn(function()
    while task.wait(900) do
        if Settings.Performance.AntiAFK then
            pcall(function()
                local vu = game:GetService("VirtualUser")
                vu:CaptureController()
                vu:ClickButton2(Vector2.new())
            end)
        end
    end
end)

local function AplicarFPSLimit(v) pcall(function() setfpscap(v) end) end

-- BUG FIX #3: re-coleta reduzida + batch de instâncias
local antiLagOriginals = {}
local antiLagActive = false
local lastAntiLagCollect = 0

local AL_TYPES = {
    ParticleEmitter = true, Trail = true, Smoke = true, Fire = true,
    Sparkles = true, Beam = true, Decal = true, Texture = true, BasePart = true,
}

local function CollectAntiLagNearby()
    local mr = GetHRP()
    if not mr then return end
    local origin = mr.Position
    -- Otimização: pega apenas primeiros níveis, não GetDescendants completo
    for _, child in ipairs(Workspace:GetChildren()) do
        if child:IsA("Model") or child:IsA("BasePart") or child:IsA("Folder") then
            local root = child:IsA("BasePart") and child or child:FindFirstChildWhichIsA("BasePart", true)
            if root and (root.Position - origin).Magnitude <= 500 then
                for _, d in ipairs(child:GetDescendants()) do
                    if AL_TYPES[d.ClassName] then
                        antiLagOriginals[d] = antiLagOriginals[d] or {}
                    end
                end
            end
        end
    end
end

local function RestoreAntiLag()
    for inst, props in pairs(antiLagOriginals) do
        if inst and inst.Parent then
            for prop, val in pairs(props) do
                pcall(function() inst[prop] = val end)
            end
        end
    end
    antiLagOriginals = {}
    if RestoreStack["__GlobalShadows"] ~= nil then
        pcall(function() Lighting.GlobalShadows = RestoreStack["__GlobalShadows"] end)
        RestoreStack["__GlobalShadows"] = nil
    end
end

local function ApplyAntiLag()
    for inst, props in pairs(antiLagOriginals) do
        if inst and inst.Parent then
            if Settings.Performance.RemoveTextures and (inst:IsA("Decal") or inst:IsA("Texture")) then
                if props.Transparency == nil then props.Transparency = inst.Transparency end
                pcall(function() inst.Transparency = 1 end)
            end
            if Settings.Performance.AntiLag and (inst:IsA("ParticleEmitter") or inst:IsA("Trail") or inst:IsA("Smoke") or inst:IsA("Fire") or inst:IsA("Sparkles") or inst:IsA("Beam")) then
                if props.Enabled == nil then props.Enabled = inst.Enabled end
                pcall(function() inst.Enabled = false end)
            end
            if Settings.Performance.RemoveShadows and inst:IsA("BasePart") then
                if props.CastShadow == nil then props.CastShadow = inst.CastShadow end
                pcall(function() inst.CastShadow = false end)
            end
        end
    end
    if Settings.Performance.RemoveShadows then
        SaveOrig("__GlobalShadows", Lighting.GlobalShadows)
        pcall(function() Lighting.GlobalShadows = false end)
    end
end

local function AplicarOtimizacao()
    local qualquer = Settings.Performance.RemoveTextures or Settings.Performance.AntiLag or Settings.Performance.RemoveShadows
    if not qualquer then
        if next(antiLagOriginals) ~= nil or RestoreStack["__GlobalShadows"] ~= nil then
            RestoreAntiLag()
        end
        antiLagActive = false
        return
    end
    if not antiLagActive or (tick() - lastAntiLagCollect) > 5 then
        antiLagOriginals = {}
        CollectAntiLagNearby()
        antiLagActive = true
        lastAntiLagCollect = tick()
    end
    ApplyAntiLag()
end
task.spawn(function()
    while Gui and Gui.Parent do
        pcall(AplicarOtimizacao)
        task.wait(4)
    end
end)

-- ===================== VEÍCULOS =====================
-- BUG FIX #2: cache de veículos (refresh a cada 1.5s, não a cada frame)
local VehicleCache, VehicleList = {}, {}
local lastVehicleScan = 0
local CHASSIS_VALUES_CACHE = nil
local lastChassisCache = 0

local function IsVehicle(obj)
    if not obj:IsA("Model") then return false end
    local mgmt = obj:FindFirstChild("Management")
    if mgmt and (mgmt:FindFirstChild("Owner") or mgmt:FindFirstChild("Vehicle")) then return true end
    if obj:GetAttribute("Vehicle") or obj:GetAttribute("Owner") or obj:GetAttribute("Gasoline") or obj:GetAttribute("Locked") then return true end
    if obj:FindFirstChild("Chassis") or obj:FindFirstChild("VehicleSeat") or obj:FindFirstChild("DriveSeat") then return true end
    return false
end

local function GetVehicles(force)
    local now = tick()
    if not force and (now - lastVehicleScan) < 1.5 and #VehicleList > 0 then
        return VehicleList
    end
    lastVehicleScan = now
    local v = {}
    for _, obj in ipairs(Workspace:GetChildren()) do
        if IsVehicle(obj) then table.insert(v, obj) end
    end
    VehicleList = v
    return v
end

local function GetMyVehicle()
    local char = LocalPlayer.Character
    if not char then return nil end
    local hum = char:FindFirstChildOfClass("Humanoid")
    if hum and hum.SeatPart then return hum.SeatPart:FindFirstAncestorOfClass("Model") end
    return nil
end

local function GetVehicleName(v)
    if not v then return "?" end
    local mgmt = v:FindFirstChild("Management")
    if mgmt then
        local nm = mgmt:FindFirstChild("Vehicle")
        if nm and nm:IsA("StringValue") then return nm.Value end
    end
    return v:GetAttribute("Name") or v.Name
end

-- BUG FIX #4: cache de ChassisValues (evita traversal por frame)
local function GetChassisValues()
    local now = tick()
    if CHASSIS_VALUES_CACHE and CHASSIS_VALUES_CACHE.Parent and (now - lastChassisCache) < 3 then
        return CHASSIS_VALUES_CACHE
    end
    lastChassisCache = now
    local path = ReplicatedStorage:FindFirstChild("shared")
    if not path then CHASSIS_VALUES_CACHE = nil; return nil end
    local storage = path:FindFirstChild("Storage"); if not storage then CHASSIS_VALUES_CACHE = nil; return nil end
    local carros = storage:FindFirstChild("Carros"); if not carros then CHASSIS_VALUES_CACHE = nil; return nil end
    local recursos = carros:FindFirstChild("Recursos"); if not recursos then CHASSIS_VALUES_CACHE = nil; return nil end
    local chUI = recursos:FindFirstChild("ChassisUI"); if not chUI then CHASSIS_VALUES_CACHE = nil; return nil end
    CHASSIS_VALUES_CACHE = chUI:FindFirstChild("Values")
    return CHASSIS_VALUES_CACHE
end

local function VehicleESPLoop()
    if not Drawing_Lib then return end
    if not Settings.Vehicle.ESPEnabled then
        for _, tbl in pairs(VehicleCache) do
            for _, o in pairs(tbl) do pcall(function() o:Remove() end) end
        end
        VehicleCache = {}
        return
    end
    local mr = GetHRP()
    if not mr then return end
    local liveV = {}
    for _, v in ipairs(GetVehicles()) do
        if v.Parent then
            liveV[v] = true
            local root = v.PrimaryPart or v:FindFirstChild("HumanoidRootPart") or v:FindFirstChildWhichIsA("BasePart")
            if root then
                local d = (root.Position - mr.Position).Magnitude
                if d <= Settings.Vehicle.ESPDistance then
                    if not VehicleCache[v] then
                        VehicleCache[v] = { box = Drawing_Lib.new("Square"), name = Drawing_Lib.new("Text") }
                        VehicleCache[v].box.Filled = false
                        VehicleCache[v].box.Thickness = 2
                    end
                    local e = VehicleCache[v]
                    local sp = Camera:WorldToViewportPoint(root.Position)
                    if sp.Z > 0 then
                        local sz = math.clamp(2000 / d, 10, 200)
                        e.box.Position = Vector2.new(sp.X - sz/2, sp.Y - sz/3)
                        e.box.Size = Vector2.new(sz, sz*0.66)
                        e.box.Color = Settings.Vehicle.ESPColor
                        e.box.Visible = true
                        e.name.Text = GetVehicleName(v).." · "..math.floor(d).."m"
                        e.name.Position = Vector2.new(sp.X, sp.Y - sz/3 - 20)
                        e.name.Color = Settings.Vehicle.ESPColor
                        e.name.Size = 12; e.name.Center = true; e.name.Outline = true; e.name.Visible = true
                    else
                        e.box.Visible = false; e.name.Visible = false
                    end
                end
            end
        end
    end
    -- BUG FIX #14: limpar veículos destruídos do cache
    for v, tbl in pairs(VehicleCache) do
        if not liveV[v] then
            for _, o in pairs(tbl) do pcall(function() o:Remove() end) end
            VehicleCache[v] = nil
        end
    end
end

task.spawn(function()
    while Gui and Gui.Parent do
        local myV = GetMyVehicle()
        if myV then
            if Settings.Vehicle.NoClip then
                for _, p in ipairs(myV:GetDescendants()) do
                    if p:IsA("BasePart") then
                        if RestoreStack[p] == nil then RestoreStack[p] = p.CanCollide end
                        pcall(function() p.CanCollide = false end)
                    end
                end
            else
                for _, p in ipairs(myV:GetDescendants()) do
                    if p:IsA("BasePart") and RestoreStack[p] ~= nil then
                        pcall(function() p.CanCollide = RestoreStack[p] end)
                        RestoreStack[p] = nil
                    end
                end
            end
            if Settings.Vehicle.InfiniteFuel then
                for _, obj in ipairs(myV:GetDescendants()) do
                    if obj:IsA("NumberValue") and (obj.Name:lower():find("gas") or obj.Name:lower():find("fuel") or obj.Name:lower():find("combust")) then
                        if RestoreStack[obj] == nil then RestoreStack[obj] = obj.Value end
                        pcall(function() obj.Value = 100 end)
                    end
                end
            end
            if Settings.Vehicle.Invincible then
                for _, obj in ipairs(myV:GetDescendants()) do
                    if obj:IsA("NumberValue") and (obj.Name:lower():find("health") or obj.Name:lower():find("vida")) then
                        if RestoreStack[obj] == nil then RestoreStack[obj] = obj.Value end
                        pcall(function() obj.Value = 9999 end)
                    end
                end
            end
            if Settings.Vehicle.EngineBoost then
                local vals = GetChassisValues()
                if vals then
                    local hp = vals:FindFirstChild("Horsepower")
                    if hp then
                        if RestoreStack[hp] == nil then RestoreStack[hp] = hp.Value end
                        pcall(function() hp.Value = Settings.Vehicle.Horsepower end)
                    end
                    local bst = vals:FindFirstChild("Boost")
                    if bst then
                        if RestoreStack[bst] == nil then RestoreStack[bst] = bst.Value end
                        pcall(function() bst.Value = Settings.Vehicle.FlyBoostMult end)
                    end
                end
            else
                local vals = GetChassisValues()
                if vals then
                    for _, nm in ipairs({"Horsepower","Boost","BoostTurbo"}) do
                        local obj = vals:FindFirstChild(nm)
                        if obj and RestoreStack[obj] ~= nil then
                            pcall(function() obj.Value = RestoreStack[obj] end)
                            RestoreStack[obj] = nil
                        end
                    end
                end
            end
            if Settings.Vehicle.AntiPIT then
                local root = myV.PrimaryPart or myV:FindFirstChildWhichIsA("BasePart")
                if root then
                    pcall(function()
                        if math.abs(root.AssemblyAngularVelocity.Y) > 5 then
                            root.AssemblyAngularVelocity = Vector3.new(root.AssemblyAngularVelocity.X * 0.1, 0, root.AssemblyAngularVelocity.Z * 0.1)
                        end
                    end)
                end
            end
        end
        task.wait(0.4)
    end
end)

task.spawn(function()
    while Gui and Gui.Parent do
        if Settings.Vehicle.Fly then
            local myV = GetMyVehicle()
            if myV then
                local root = myV.PrimaryPart or myV:FindFirstChildWhichIsA("BasePart")
                if root then
                    local delta = RunService.RenderStepped:Wait()
                    local vel = Vector3.new()
                    local spd = Settings.Vehicle.FlySpeed * delta
                    local cam = Camera.CFrame
                    if K_W and UserInputService:IsKeyDown(K_W) then vel = vel + cam.LookVector end
                    if K_S and UserInputService:IsKeyDown(K_S) then vel = vel - cam.LookVector end
                    if K_A and UserInputService:IsKeyDown(K_A) then vel = vel - cam.RightVector end
                    if K_D and UserInputService:IsKeyDown(K_D) then vel = vel + cam.RightVector end
                    if K_SPACE and UserInputService:IsKeyDown(K_SPACE) then vel = vel + Vector3.new(0,1,0) end
                    if K_LCTRL and UserInputService:IsKeyDown(K_LCTRL) then vel = vel - Vector3.new(0,1,0) end
                    if vel.Magnitude > 0 then
                        vel = vel.Unit * spd * Settings.Vehicle.FlyAccel
                        pcall(function() root.CFrame = root.CFrame + vel end)
                        pcall(function() root.AssemblyLinearVelocity = vel end)
                    end
                end
            else
                task.wait(0.1)
            end
        else
            task.wait(0.1)
        end
    end
end)

task.spawn(function()
    while Gui and Gui.Parent do
        if Settings.Vehicle.Fling and Settings.Vehicle.SelectedVehicle then
            local target = Settings.Vehicle.SelectedVehicle
            local tRoot = target.PrimaryPart or target:FindFirstChildWhichIsA("BasePart")
            if tRoot then
                tRoot.AssemblyLinearVelocity = Vector3.new(999, 999, 999)
                tRoot.AssemblyAngularVelocity = Vector3.new(999, 999, 999)
            end
        end
        task.wait(0.1)
    end
end)

local function FindTrunk(myV)
    if not myV then return nil end
    local mgmt = myV:FindFirstChild("Management")
    if mgmt then
        local t = mgmt:FindFirstChild("Trunk") or mgmt:FindFirstChild("PortaMalas")
        if t then return t end
    end
    return myV:FindFirstChild("Trunk") or myV:FindFirstChild("PortaMalas")
        or myV:FindFirstChild("Porta-Malas") or myV:FindFirstChild("Malas")
end

local function RefreshTrunkItems()
    if S.TrunkItemsLabel then S.TrunkItemsLabel.Text = "None" end
    local myV = GetMyVehicle()
    if not myV then Notify("Sem veículo", RED, "X"); return end
    local trunk = FindTrunk(myV)
    if not trunk then Notify("Porta-malas não localizável (server-side)", ORANGE, "⚠️"); return end
    local items = {}
    for _, c in ipairs(trunk:GetChildren()) do
        if c:IsA("Tool") or c:IsA("Model") then table.insert(items, c.Name) end
    end
    if S.TrunkItemsLabel then S.TrunkItemsLabel.Text = (#items > 0) and (#items.." itens") or "None" end
    Settings.Vehicle.TrunkItems = items
    Notify("Itens atualizados: "..#items, GREEN, "✓")
end

-- BUG FIX #6: AplicarAcaoTrunk não chama remotes sem schema (dossiê: push-only)
local function AplicarAcaoTrunk(acao)
    Notify("⚠️ Ação '"..acao.."' indisponível — remotes do servidor não expõem esse comando (dossiê push-only)", ORANGE, "🛡")
end

local function EntrarVeiculo(veh)
    if not veh or not veh.Parent then Notify("Nenhum veículo selecionado", RED, "X"); return end
    local mr = GetHRP()
    local tRoot = veh.PrimaryPart or veh:FindFirstChildWhichIsA("BasePart")
    if mr and tRoot then
        mr.CFrame = tRoot.CFrame
        Notify("Teleportado ao veículo", GREEN, "✓")
    end
end

-- BUG FIX #8: TrocarEmprego removido (server-side, args desconhecidos)
local function TrocarEmprego(nome)
    Notify("⚠️ Trocar emprego é server-side — sem args expostos", ORANGE, "🛡")
end

-- ===================== LOOP PRINCIPAL =====================
RunService:BindToRenderStep("TropaAim", Enum.RenderPriority.Camera.Value + 1, function() pcall(AimbotLoop) end)
RunService.RenderStepped:Connect(function()
    pcall(TriggerLoop)
    pcall(ESPLoop)
    pcall(UpdateChams)
    pcall(VehicleESPLoop)
    pcall(NoRecoilLoop)
    pcall(GodLoop)
    pcall(ReviveLoop)
    pcall(MoveLoop)
    pcall(ColarLoop)
    if Config.FOVAtivo then pcall(function() Camera.FieldOfView = Config.FOV_Camera end) end
end)

-- ===================== UI COMPONENTS =====================
local function Section(parent, title, sub)
    local f = Instance.new("Frame")
    f.Size = UDim2.new(1,0,0,56); f.BackgroundTransparency = 1; f.Parent = parent
    local barra = Instance.new("Frame")
    barra.Size = UDim2.fromOffset(3,40); barra.Position = UDim2.fromOffset(0,6)
    barra.BackgroundColor3 = PURPLE; barra.BorderSizePixel = 0
    barra.Parent = f; Corner(barra, 2)
    Gradient(barra, PURPLE, PURPLE_LT, 90)
    Lbl(f, title, 20, UDim2.fromOffset(14,0), Enum.Font.GothamBlack).Size = UDim2.new(1,-14,0,30)
    Lbl(f, sub or "", 11, UDim2.fromOffset(14,30), Enum.Font.Gotham, TXT_DIM).Size = UDim2.new(1,-14,0,18)
    f.Name = "Section"
end

local function Card(parent, title, corTopo)
    corTopo = corTopo or PURPLE
    local c = Instance.new("Frame")
    c.Size = UDim2.new(1,0,0,0); c.AutomaticSize = Enum.AutomaticSize.Y
    c.BackgroundColor3 = CARD_BG
    c.Parent = parent; Corner(c, 14)
    local st = Stroke(c, BORDER, 1)
    Gradient(c, CARD_BG, BG, 135)
    local topBar = Instance.new("Frame")
    topBar.Size = UDim2.new(1,-32,0,2); topBar.Position = UDim2.fromOffset(16,0)
    topBar.BackgroundColor3 = Color3.new(1,1,1); topBar.BorderSizePixel = 0
    topBar.Parent = c
    local tbg = Gradient(topBar, corTopo, PURPLE_LT, 0)
    tbg.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 1), NumberSequenceKeypoint.new(0.5, 0), NumberSequenceKeypoint.new(1, 1),
    })
    local pad = Instance.new("UIPadding", c)
    pad.PaddingTop = UDim.new(0,16); pad.PaddingBottom = UDim.new(0,16)
    pad.PaddingLeft = UDim.new(0,18); pad.PaddingRight = UDim.new(0,18)
    local lay = Instance.new("UIListLayout", c)
    lay.Padding = UDim.new(0,5); lay.SortOrder = Enum.SortOrder.LayoutOrder
    if title and title ~= "" then
        local t = Instance.new("TextLabel", c)
        t.Size = UDim2.new(1,0,0,24); t.BackgroundTransparency = 1
        t.Text = title; t.TextColor3 = TXT; t.TextSize = 13
        t.Font = Enum.Font.GothamBold; t.TextXAlignment = Enum.TextXAlignment.Left
        t.LayoutOrder = -1
    end
    c.MouseEnter:Connect(function()
        Tw(c, {BackgroundColor3 = CARD_HOVER}, 0.22)
        Tw(st, {Color = PURPLE, Thickness = 1.5}, 0.22)
    end)
    c.MouseLeave:Connect(function()
        Tw(c, {BackgroundColor3 = CARD_BG}, 0.22)
        Tw(st, {Color = BORDER, Thickness = 1}, 0.22)
    end)
    return c
end

local function Toggle(parent, title, desc, tab, key, cb, danger)
    local row = Instance.new("Frame")
    row.Size = UDim2.new(1,0,0,52); row.BackgroundTransparency = 1; row.Parent = parent
    local descTxt = desc or ""
    if danger then descTxt = "⚠️ "..descTxt.." — RISCO DE KICK" end
    local lblTit = Lbl(row, title, 13, UDim2.fromOffset(0,4), Enum.Font.GothamMedium)
    lblTit.Size = UDim2.new(1,-80,0,18)
    if danger then lblTit.TextColor3 = ORANGE end
    local lblDesc = Lbl(row, descTxt, 11, UDim2.fromOffset(0,24), Enum.Font.Gotham, danger and ORANGE or TXT_DIM)
    lblDesc.Size = UDim2.new(1,-80,0,18)
    local sw = Instance.new("TextButton")
    sw.Size = UDim2.fromOffset(46,26); sw.Position = UDim2.new(1,-46,0.5,-13)
    sw.BackgroundColor3 = tab[key] and PURPLE or SWITCH_OFF
    sw.Text = ""; sw.AutoButtonColor = false; sw.Parent = row
    Corner(sw, 13)
    local swStroke = Stroke(sw, tab[key] and PURPLE_LT or BORDER, 1)
    local knob = Instance.new("Frame")
    knob.Size = UDim2.fromOffset(20,20)
    knob.Position = tab[key] and UDim2.fromOffset(23,3) or UDim2.fromOffset(3,3)
    knob.BackgroundColor3 = Color3.new(1,1,1); knob.BorderSizePixel = 0
    knob.Parent = sw; Corner(knob, 10)
    local function Update()
        if tab[key] then
            Tw(sw, {BackgroundColor3 = danger and ORANGE or PURPLE}, 0.2)
            Tw(swStroke, {Color = danger and ORANGE or PURPLE_LT}, 0.2)
            Tw(knob, {Position = UDim2.fromOffset(23,3)}, 0.2, Enum.EasingStyle.Back)
        else
            Tw(sw, {BackgroundColor3 = SWITCH_OFF}, 0.2)
            Tw(swStroke, {Color = BORDER}, 0.2)
            Tw(knob, {Position = UDim2.fromOffset(3,3)}, 0.2, Enum.EasingStyle.Back)
        end
    end
    sw.MouseButton1Click:Connect(function()
        tab[key] = not tab[key]
        Tw(sw, {Size = UDim2.fromOffset(54,32), Position = UDim2.new(1,-49,0.5,-16)}, 0.1)
        task.delay(0.1, function()
            Tw(sw, {Size = UDim2.fromOffset(46,26), Position = UDim2.new(1,-46,0.5,-13)}, 0.18, Enum.EasingStyle.Back)
        end)
        Ripple(sw); Update()
        if cb then cb(tab[key]) end
    end)
    Update()
end

local function Slider(parent, title, min, max, tab, key, cb)
    local row = Instance.new("Frame")
    row.Size = UDim2.new(1,0,0,70); row.BackgroundTransparency = 1; row.Parent = parent
    Lbl(row, title, 13, UDim2.fromOffset(0,4), Enum.Font.GothamMedium).Size = UDim2.new(0.6,0,0,18)
    local valBox = Instance.new("TextBox")
    valBox.Size = UDim2.fromOffset(72,28); valBox.Position = UDim2.new(1,-72,0,0)
    valBox.BackgroundColor3 = INPUT_BG
    valBox.Text = tostring(tab[key]); valBox.TextColor3 = PURPLE_LT
    valBox.TextSize = 13; valBox.Font = Enum.Font.GothamBold
    valBox.ClearTextOnFocus = false; valBox.Parent = row
    Corner(valBox, 8); Stroke(valBox, BORDER)
    local track = Instance.new("Frame")
    track.Size = UDim2.new(1,0,0,8); track.Position = UDim2.fromOffset(0,42)
    track.BackgroundColor3 = Color3.fromRGB(50,50,68); track.BorderSizePixel = 0
    track.Parent = row; Corner(track, 4)
    local pct = math.clamp((tab[key] - min) / (max - min), 0, 1)
    local fill = Instance.new("Frame")
    fill.Size = UDim2.new(pct,0,1,0); fill.BackgroundColor3 = PURPLE
    fill.BorderSizePixel = 0; fill.Parent = track; Corner(fill, 4)
    Gradient(fill, PURPLE, PURPLE_LT, 0)
    local balao = Instance.new("TextLabel")
    balao.Size = UDim2.fromOffset(54,24); balao.AnchorPoint = Vector2.new(0.5,1)
    balao.Position = UDim2.new(pct,0,0,-6); balao.BackgroundColor3 = PURPLE
    balao.Text = tostring(tab[key]); balao.TextColor3 = Color3.new(1,1,1)
    balao.TextSize = 11; balao.Font = Enum.Font.GothamBold
    balao.Visible = false; balao.ZIndex = 20; balao.Parent = track; Corner(balao, 6)
    local knob = Instance.new("Frame")
    knob.Size = UDim2.fromOffset(22,22); knob.AnchorPoint = Vector2.new(0.5,0.5)
    knob.Position = UDim2.new(pct,0,0.5,0); knob.BackgroundColor3 = Color3.new(1,1,1)
    knob.BorderSizePixel = 0; knob.Parent = track; Corner(knob, 11); Stroke(knob, PURPLE, 2)
    local knobBtn = Instance.new("TextButton")
    knobBtn.Size = UDim2.new(1,0,1,0); knobBtn.BackgroundTransparency = 1
    knobBtn.Text = ""; knobBtn.AutoButtonColor = false; knobBtn.Parent = knob
    local dragging = false
    local lastCbCall = 0
    local function setFromX(x, fromRelease)
        local p2 = math.clamp((x - track.AbsolutePosition.X) / math.max(track.AbsoluteSize.X, 1), 0, 1)
        local v = min + (max - min) * p2
        if max <= 5 and min < 1 then v = math.floor(v*10+0.5)/10 else v = math.floor(v+0.5) end
        tab[key] = v
        fill.Size = UDim2.new(p2,0,1,0)
        knob.Position = UDim2.new(p2,0,0.5,0)
        balao.Position = UDim2.new(p2,0,0,-6); balao.Text = tostring(v)
        if not valBox:IsFocused() then valBox.Text = tostring(v) end
        if cb then
            local now = tick()
            if fromRelease or (now - lastCbCall) > 0.15 then
                lastCbCall = now
                cb(v)
            end
        end
    end
    local function startDrag(x) dragging=true; balao.Visible=true; Tw(knob,{Size=UDim2.fromOffset(28,28)},0.15); setFromX(x) end
    local function stopDrag()
        dragging=false; balao.Visible=false
        Tw(knob,{Size=UDim2.fromOffset(22,22)},0.15)
        if cb then cb(tab[key]) end
    end
    track.InputBegan:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 then startDrag(i.Position.X) end end)
    knobBtn.InputBegan:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 then startDrag(i.Position.X) end end)
    UserInputService.InputChanged:Connect(function(i)
        if dragging and i.UserInputType == Enum.UserInputType.MouseMovement then setFromX(i.Position.X) end
    end)
    UserInputService.InputEnded:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 and dragging then stopDrag() end
    end)
    valBox.FocusLost:Connect(function()
        local n = tonumber(valBox.Text)
        if n then
            n = math.clamp(n, min, max)
            tab[key] = n
            local p2 = (n - min) / (max - min)
            fill.Size = UDim2.new(p2,0,1,0)
            knob.Position = UDim2.new(p2,0,0.5,0)
            valBox.Text = tostring(n)
            if cb then cb(n) end
        else valBox.Text = tostring(tab[key]) end
    end)
end

local function BigBtn(parent, txt, cb, danger)
    local b = Instance.new("TextButton")
    b.Size = UDim2.new(1,0,0,46)
    b.BackgroundColor3 = danger and Color3.fromRGB(70,20,30) or INPUT_BG
    b.Text = txt; b.TextColor3 = Color3.new(1,1,1); b.TextSize = 13
    b.Font = Enum.Font.GothamBold; b.AutoButtonColor = false
    b.ClipsDescendants = true; b.Parent = parent; Corner(b, 12)
    local bst = Stroke(b, danger and RED or PURPLE, 1.5)
    if not danger then Gradient(b, INPUT_BG, Color3.fromRGB(22,22,32), 90) end
    local shine = Instance.new("Frame")
    shine.Size = UDim2.new(1,-20,0,1); shine.Position = UDim2.fromOffset(10,1)
    shine.BackgroundColor3 = Color3.fromRGB(120,120,150); shine.BackgroundTransparency = 0.5
    shine.BorderSizePixel = 0; shine.Parent = b; Corner(shine, 1)
    HookHover(b, danger and Color3.fromRGB(70,20,30) or INPUT_BG,
        danger and Color3.fromRGB(100,30,45) or CARD_HOVER,
        bst, danger and RED or PURPLE_LT)
    b.MouseButton1Down:Connect(function() Ripple(b) end)
    b.MouseButton1Click:Connect(function() if cb then cb() end end)
    return b
end

local function Dropdown(parent, title, options, tab, key, cb)
    local row = Instance.new("Frame")
    row.Size = UDim2.new(1,0,0,46); row.BackgroundTransparency = 1; row.Parent = parent
    Lbl(row, title, 13, UDim2.fromOffset(0,14), Enum.Font.GothamMedium).Size = UDim2.new(0.5,0,0,18)
    local sel = Instance.new("TextButton")
    sel.Size = UDim2.fromOffset(140,34); sel.Position = UDim2.new(1,-140,0.5,-17)
    sel.BackgroundColor3 = INPUT_BG
    sel.Text = tostring(tab[key]); sel.TextColor3 = PURPLE_LT
    sel.TextSize = 11; sel.Font = Enum.Font.GothamBold
    sel.AutoButtonColor = false; sel.Parent = row
    Corner(sel, 10)
    local sst = Stroke(sel, BORDER)
    Gradient(sel, INPUT_BG, BG, 90)
    HookHover(sel, INPUT_BG, CARD_HOVER, sst, PURPLE_LT)
    local idx = 1
    for i, o in ipairs(options) do if o == tab[key] then idx = i; break end end
    sel.MouseButton1Click:Connect(function()
        idx = idx % #options + 1
        tab[key] = options[idx]
        sel.Text = tostring(options[idx])
        if cb then cb(options[idx]) end
    end)
end

local function InputRow(parent, title, tab, key, validator)
    local row = Instance.new("Frame")
    row.Size = UDim2.new(1,0,0,46); row.BackgroundTransparency = 1; row.Parent = parent
    Lbl(row, title, 13, UDim2.fromOffset(0,14), Enum.Font.GothamMedium).Size = UDim2.new(0.5,0,0,18)
    local ip = Instance.new("TextBox")
    ip.Size = UDim2.fromOffset(84,34); ip.Position = UDim2.new(1,-84,0.5,-17)
    ip.BackgroundColor3 = INPUT_BG
    ip.Text = tostring(tab[key]); ip.TextColor3 = PURPLE_LT
    ip.TextSize = 13; ip.Font = Enum.Font.GothamBold
    ip.ClearTextOnFocus = false; ip.Parent = row
    Corner(ip, 10)
    local ipst = Stroke(ip, BORDER)
    ip.FocusLost:Connect(function()
        if validator then
            local ok, v = validator(ip.Text)
            if ok then tab[key] = v else ip.Text = tostring(tab[key]) end
        else tab[key] = ip.Text end
        Tw(ipst, {Color = BORDER}, 0.15)
    end)
    ip.Focused:Connect(function() Tw(ipst, {Color = PURPLE_LT}, 0.15) end)
end

local function ColorRow(parent, title, tab, key, corInicial)
    local row = Instance.new("Frame")
    row.Size = UDim2.new(1,0,0,46); row.BackgroundTransparency = 1; row.Parent = parent
    Lbl(row, title, 13, UDim2.fromOffset(0,14), Enum.Font.GothamMedium).Size = UDim2.new(0.5,0,0,18)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.fromOffset(80,28); btn.Position = UDim2.new(1,-80,0.5,-14)
    btn.BackgroundColor3 = corInicial or tab[key]
    btn.Text = ""; btn.AutoButtonColor = false; btn.Parent = row
    Corner(btn, 8); Stroke(btn, BORDER)
    local palette = {
        Color3.fromRGB(0,230,140), Color3.fromRGB(200,80,255), Color3.fromRGB(80,160,255),
        Color3.fromRGB(255,90,90), Color3.fromRGB(255,200,60), Color3.fromRGB(255,255,255),
        Color3.fromRGB(255,150,200),
    }
    local idx = 1
    btn.MouseButton1Click:Connect(function()
        idx = idx % #palette + 1
        tab[key] = palette[idx]
        btn.BackgroundColor3 = palette[idx]
    end)
end

local function AplicarBusca(txt)
    txt = txt:lower()
    for _, pg in pairs(S.Pages) do
        for _, child in ipairs(pg:GetChildren()) do
            if child:IsA("Frame") then
                if child.Name == "Section" then child.Visible = true
                else
                    local mostra = true
                    if txt ~= "" then
                        mostra = false
                        for _, d in ipairs(child:GetDescendants()) do
                            if d:IsA("TextLabel") and d.Text:lower():find(txt, 1, true) then
                                mostra = true; break
                            end
                        end
                    end
                    child.Visible = mostra
                end
            end
        end
    end
end

local function SelectTab(name)
    for n, b in pairs(S.TabBtns) do
        if n == name then
            Tw(b.btn, {BackgroundColor3 = Color3.fromRGB(50,25,75)}, 0.22)
            Tw(b.stroke, {Color = PURPLE, Thickness = 1.5}, 0.22)
            b.sub.TextColor3 = PURPLE_LT
            Tw(b.indicator, {Size = UDim2.new(0,4,1,-16), Position = UDim2.fromOffset(0,8)}, 0.22)
        else
            Tw(b.btn, {BackgroundColor3 = SIDE_BG}, 0.22)
            Tw(b.stroke, {Color = BORDER, Thickness = 1}, 0.22)
            b.sub.TextColor3 = TXT_DIM
            Tw(b.indicator, {Size = UDim2.new(0,0,1,-16)}, 0.22)
        end
    end
    for n, pg in pairs(S.Pages) do
        if n == name then
            pg.Visible = true
            pg.Position = UDim2.fromOffset(24,0)
            Tw(pg, {Position = UDim2.fromOffset(0,0)}, 0.32)
        else pg.Visible = false end
    end
end

-- ===================== LOGIN =====================
local function BuildLogin()
    local Login = Instance.new("Frame")
    Login.Size = UDim2.fromScale(1,1); Login.BackgroundTransparency = 1
    Login.BorderSizePixel = 0; Login.ZIndex = 10; Login.Parent = Gui

    local Card_ = Instance.new("Frame")
    Card_.Size = UDim2.fromOffset(420, 460)
    Card_.Position = UDim2.fromScale(0.5, 0.5)
    Card_.AnchorPoint = Vector2.new(0.5, 0.5)
    Card_.BackgroundColor3 = Color3.fromRGB(18, 18, 26)
    Card_.ClipsDescendants = true
    Card_.Parent = Login
    Corner(Card_, 24); Stroke(Card_, BORDER, 1.5)
    Gradient(Card_, Color3.fromRGB(26,26,38), Color3.fromRGB(10,10,18), 135)
    Glow(Card_, PURPLE, 1.2)

    local AvatarWrap = Instance.new("Frame")
    AvatarWrap.Size = UDim2.fromOffset(110, 110)
    AvatarWrap.Position = UDim2.new(0.5, -55, 0, 34)
    AvatarWrap.BackgroundTransparency = 1; AvatarWrap.Parent = Card_
    local glowBg = Instance.new("Frame")
    glowBg.Size = UDim2.fromScale(1,1); glowBg.BackgroundColor3 = PURPLE
    glowBg.BackgroundTransparency = 0.85; glowBg.BorderSizePixel = 0
    glowBg.Parent = AvatarWrap; Corner(glowBg, 55)
    Pulse(glowBg, "BackgroundTransparency", 0.85, 0.7, 2.5)
    local anel = Instance.new("Frame")
    anel.Size = UDim2.fromScale(1, 1); anel.BackgroundTransparency = 1; anel.Parent = AvatarWrap
    local anelS = Stroke(anel, PURPLE, 2, 0.15); AddRGB(anelS)
    Corner(anel, 100); Pulse(anel, "Rotation", 0, 360, 10)
    local AvatarFrame = Instance.new("Frame")
    AvatarFrame.Size = UDim2.fromOffset(84, 84)
    AvatarFrame.Position = UDim2.new(0.5, -42, 0.5, -42)
    AvatarFrame.BackgroundColor3 = Color3.fromRGB(25, 10, 40)
    AvatarFrame.Parent = AvatarWrap; Corner(AvatarFrame, 42)
    Stroke(AvatarFrame, PURPLE_LT, 1.5, 0.2)
    Gradient(AvatarFrame, Color3.fromRGB(50,20,80), Color3.fromRGB(15,5,28), 45)
    local AvatarImg = Instance.new("ImageLabel")
    AvatarImg.Size = UDim2.fromScale(1,1); AvatarImg.BackgroundTransparency = 1
    AvatarImg.Image = ""; AvatarImg.Parent = AvatarFrame; Corner(AvatarImg, 42)
    task.spawn(function()
        local ok, img = pcall(function()
            return Players:GetUserThumbnailAsync(LocalPlayer.UserId, Enum.ThumbnailType.HeadShot, Enum.ThumbnailSize.Size150x150)
        end)
        if ok and img then AvatarImg.Image = img end
    end)
    local playerName = Lbl(Card_, "@"..LocalPlayer.Name, 11, UDim2.fromOffset(0, 156), Enum.Font.GothamMedium, TXT)
    playerName.Size = UDim2.new(1, 0, 0, 16); playerName.TextXAlignment = Enum.TextXAlignment.Center
    local LTitle = Lbl(Card_, "TROPA DO LKK", 26, UDim2.fromOffset(0, 178), Enum.Font.GothamBlack, TXT)
    LTitle.Size = UDim2.new(1, 0, 0, 32); LTitle.TextXAlignment = Enum.TextXAlignment.Center
    local LSub = Lbl(Card_, "LOAD-SAFE · v6.6.0", 9, UDim2.fromOffset(0, 212), Enum.Font.GothamBold, GREEN)
    LSub.Size = UDim2.new(1, 0, 0, 14); LSub.TextXAlignment = Enum.TextXAlignment.Center
    local linha = Instance.new("Frame")
    linha.Size = UDim2.fromOffset(180, 1); linha.Position = UDim2.new(0.5, -90, 0, 234)
    linha.BackgroundColor3 = PURPLE; linha.BackgroundTransparency = 0.4
    linha.BorderSizePixel = 0; linha.Parent = Card_
    local InputWrap = Instance.new("Frame")
    InputWrap.Size = UDim2.new(1, -80, 0, 52); InputWrap.Position = UDim2.fromOffset(40, 262)
    InputWrap.BackgroundColor3 = INPUT_BG; InputWrap.Parent = Card_
    Corner(InputWrap, 14); local InputS = Stroke(InputWrap, BORDER, 1.5)
    local Ico = Instance.new("TextLabel")
    Ico.Size = UDim2.fromOffset(28, 28); Ico.Position = UDim2.fromOffset(16, 12)
    Ico.BackgroundTransparency = 1; Ico.Text = "🔒"; Ico.TextSize = 15; Ico.Parent = InputWrap
    local PassInput = Instance.new("TextBox")
    PassInput.Size = UDim2.new(1, -100, 1, 0); PassInput.Position = UDim2.fromOffset(48, 0)
    PassInput.BackgroundTransparency = 1
    PassInput.PlaceholderText = "Digite a senha..."; PassInput.PlaceholderColor3 = TXT_DIM
    PassInput.TextColor3 = TXT; PassInput.TextSize = 14
    PassInput.Font = Enum.Font.GothamMedium; PassInput.ClearTextOnFocus = false
    PassInput.Text = ""; PassInput.TextXAlignment = Enum.TextXAlignment.Left
    PassInput.Parent = InputWrap
    PassInput.Focused:Connect(function()
        Tw(InputWrap, {BackgroundColor3 = Color3.fromRGB(45,25,65)}, 0.2)
        Tw(InputS, {Color = PURPLE_LT}, 0.2)
    end)
    PassInput.FocusLost:Connect(function()
        Tw(InputWrap, {BackgroundColor3 = INPUT_BG}, 0.2)
        Tw(InputS, {Color = BORDER}, 0.2)
    end)
    local EnterBtn = Instance.new("TextButton")
    EnterBtn.Size = UDim2.new(1, -80, 0, 52); EnterBtn.Position = UDim2.fromOffset(40, 328)
    EnterBtn.BackgroundColor3 = PURPLE
    EnterBtn.Text = "ENTRAR"; EnterBtn.TextColor3 = Color3.new(1,1,1)
    EnterBtn.TextSize = 15; EnterBtn.Font = Enum.Font.GothamBold
    EnterBtn.AutoButtonColor = false; EnterBtn.ClipsDescendants = true
    EnterBtn.Parent = Card_
    Corner(EnterBtn, 14); Gradient(EnterBtn, PURPLE, PURPLE_DK, 90)
    Glow(EnterBtn, PURPLE, 0.8); Shimmer(EnterBtn)
    EnterBtn.MouseEnter:Connect(function() Tw(EnterBtn, {BackgroundColor3 = PURPLE_LT}, 0.2) end)
    EnterBtn.MouseLeave:Connect(function() Tw(EnterBtn, {BackgroundColor3 = PURPLE}, 0.2) end)
    EnterBtn.MouseButton1Down:Connect(function() Ripple(EnterBtn) end)
    local ErrorLbl = Lbl(Card_, "", 11, UDim2.fromOffset(40, 392), Enum.Font.GothamMedium, RED)
    ErrorLbl.Size = UDim2.new(1, -80, 0, 16); ErrorLbl.TextXAlignment = Enum.TextXAlignment.Center
    local footer = Lbl(Card_, "v6.6.0 · Kelp Fusion · FIXED", 9, UDim2.new(0, 0, 1, -28), Enum.Font.Gotham, GREEN)
    footer.Size = UDim2.new(1, 0, 0, 16); footer.TextXAlignment = Enum.TextXAlignment.Center
    S.Login = Login; S.LoginCard = Card_
    S.PassInput = PassInput; S.EnterBtn = EnterBtn
    S.ErrorLbl = ErrorLbl; S.InputWrap = InputWrap
end

-- ===================== PANEL =====================
local function BuildPanel()
    local Panel = Instance.new("Frame")
    Panel.Size = UDim2.fromOffset(1000,600)
    Panel.Position = UDim2.fromScale(0.5,0.5); Panel.AnchorPoint = Vector2.new(0.5,0.5)
    Panel.BackgroundColor3 = Color3.fromRGB(12,12,18); Panel.BorderSizePixel = 0
    Panel.Visible = false; Panel.Active = true
    Panel.ClipsDescendants = true; Panel.Parent = Gui
    Corner(Panel, CORNER_R); Stroke(Panel, BORDER, 1.5)
    Gradient(Panel, Color3.fromRGB(18,18,28), BG, 135)
    S.Panel = Panel

    local Header = Instance.new("Frame")
    Header.Name = "Header"; Header.Size = UDim2.new(1,0,0,68)
    Header.BackgroundColor3 = Color3.fromRGB(18,18,26); Header.BorderSizePixel = 0
    Header.ClipsDescendants = true
    Header.Parent = Panel; Corner(Header, CORNER_R)
    Gradient(Header, Color3.fromRGB(24,24,34), Color3.fromRGB(14,14,22), 90)
    local HL = Instance.new("Frame")
    HL.Size = UDim2.new(1,0,0,1); HL.Position = UDim2.new(0,0,1,-1)
    HL.BackgroundColor3 = BORDER; HL.BorderSizePixel = 0
    HL.Parent = Header; S.Header = Header

    local Ham = Instance.new("TextButton")
    Ham.Size = UDim2.fromOffset(40,40); Ham.Position = UDim2.fromOffset(16,14)
    Ham.BackgroundColor3 = INPUT_BG
    Ham.Text = "☰"; Ham.TextColor3 = TXT; Ham.TextSize = 20
    Ham.Font = Enum.Font.GothamBold; Ham.AutoButtonColor = false
    Ham.Parent = Header; Corner(Ham, 12)
    local HamS = Stroke(Ham, BORDER)
    HookHover(Ham, INPUT_BG, CARD_HOVER, HamS, PURPLE)
    Ham.MouseButton1Down:Connect(function() Ripple(Ham) end)

    local TW = Instance.new("Frame")
    TW.Size = UDim2.fromOffset(300,44); TW.Position = UDim2.fromOffset(70,12)
    TW.BackgroundTransparency = 1; TW.Parent = Header
    Lbl(TW, "TROPA DO LKK · v6.6.0", 17, UDim2.fromOffset(0,0), Enum.Font.GothamBlack, TXT).Size = UDim2.new(1,0,0,22)
    local SR = Instance.new("Frame")
    SR.Size = UDim2.new(1,0,0,16); SR.Position = UDim2.fromOffset(0,22)
    SR.BackgroundTransparency = 1; SR.Parent = TW
    local sd = Instance.new("Frame")
    sd.Size = UDim2.fromOffset(7,7); sd.Position = UDim2.fromOffset(0,5)
    sd.BackgroundColor3 = GREEN; sd.BorderSizePixel = 0; sd.Parent = SR
    Corner(sd, 4); Pulse(sd, "BackgroundTransparency", 0, 0.6, 1)
    Lbl(SR, "FIXED · anti-kick ativo", 9, UDim2.fromOffset(14,0), Enum.Font.Gotham, GREEN).Size = UDim2.new(1,-14,1,0)

    local Clock = Lbl(Header, "00:00:00", 12, UDim2.new(1,-440,0,24), Enum.Font.GothamBold, PURPLE_LT)
    Clock.Size = UDim2.fromOffset(90,20); Clock.TextXAlignment = Enum.TextXAlignment.Right
    task.spawn(function() while Gui and Gui.Parent do Clock.Text = os.date("%H:%M:%S"); task.wait(1) end end)

    local SearchBox = Instance.new("TextBox")
    SearchBox.Size = UDim2.fromOffset(160,36); SearchBox.Position = UDim2.new(1,-340,0.5,-18)
    SearchBox.BackgroundColor3 = INPUT_BG
    SearchBox.PlaceholderText = "🔍  Buscar..."; SearchBox.PlaceholderColor3 = TXT_DIM
    SearchBox.Text = ""; SearchBox.TextColor3 = TXT; SearchBox.TextSize = 12
    SearchBox.Font = Enum.Font.Gotham; SearchBox.ClearTextOnFocus = false
    SearchBox.Parent = Header; Corner(SearchBox, 18); Stroke(SearchBox, BORDER)
    S.SearchBox = SearchBox

    local MinBtn = Instance.new("TextButton")
    MinBtn.Size = UDim2.fromOffset(34,34); MinBtn.Position = UDim2.new(1,-90,0.5,-17)
    MinBtn.BackgroundColor3 = INPUT_BG
    MinBtn.Text = "−"; MinBtn.TextColor3 = TXT; MinBtn.TextSize = 18
    MinBtn.Font = Enum.Font.GothamBold; MinBtn.AutoButtonColor = false
    MinBtn.Parent = Header; Corner(MinBtn, 10)
    local MinS = Stroke(MinBtn, BORDER)
    HookHover(MinBtn, INPUT_BG, CARD_HOVER, MinS, PURPLE_LT)

    local CloseBtn = Instance.new("TextButton")
    CloseBtn.Size = UDim2.fromOffset(34,34); CloseBtn.Position = UDim2.new(1,-50,0.5,-17)
    CloseBtn.BackgroundColor3 = INPUT_BG
    CloseBtn.Text = "✕"; CloseBtn.TextColor3 = TXT; CloseBtn.TextSize = 14
    CloseBtn.Font = Enum.Font.GothamBold; CloseBtn.AutoButtonColor = false
    CloseBtn.Parent = Header; Corner(CloseBtn, 10)
    local CloseS = Stroke(CloseBtn, BORDER)
    HookHover(CloseBtn, INPUT_BG, Color3.fromRGB(70,20,30), CloseS, RED)

    MinBtn.MouseButton1Click:Connect(function() Panel.Visible = false end)
    CloseBtn.MouseButton1Click:Connect(function() Panel.Visible = false end)

    local SW = 220
    local Sidebar = Instance.new("Frame")
    Sidebar.Size = UDim2.new(0,SW,1,-68); Sidebar.Position = UDim2.fromOffset(0,68)
    Sidebar.BackgroundColor3 = SIDE_BG
    Sidebar.BorderSizePixel = 0; Sidebar.ClipsDescendants = true
    Sidebar.Parent = Panel; Corner(Sidebar, CORNER_R)
    Gradient(Sidebar, SIDE_BG_TOP, BG, 90)
    local SS = Instance.new("ScrollingFrame")
    SS.Size = UDim2.new(1,0,1,0); SS.BackgroundTransparency = 1
    SS.BorderSizePixel = 0; SS.ScrollBarThickness = 2
    SS.ScrollBarImageColor3 = PURPLE; SS.AutomaticCanvasSize = Enum.AutomaticSize.Y
    SS.CanvasSize = UDim2.new(); SS.Parent = Sidebar
    local SL = Instance.new("UIListLayout", SS)
    SL.Padding = UDim.new(0,5); SL.SortOrder = Enum.SortOrder.LayoutOrder
    local SPad = Instance.new("UIPadding", SS)
    SPad.PaddingTop = UDim.new(0,16); SPad.PaddingLeft = UDim.new(0,12)
    SPad.PaddingRight = UDim.new(0,12); SPad.PaddingBottom = UDim.new(0,14)

    local Content = Instance.new("ScrollingFrame")
    Content.Size = UDim2.new(1,-SW,1,-68); Content.Position = UDim2.fromOffset(SW,68)
    Content.BackgroundTransparency = 1; Content.BorderSizePixel = 0
    Content.ScrollBarThickness = 3; Content.ScrollBarImageColor3 = PURPLE
    Content.AutomaticCanvasSize = Enum.AutomaticSize.Y; Content.CanvasSize = UDim2.new()
    Content.ClipsDescendants = true; Content.Parent = Panel; Corner(Content, CORNER_R)
    local CP = Instance.new("UIPadding", Content)
    CP.PaddingTop = UDim.new(0,24); CP.PaddingLeft = UDim.new(0,22)
    CP.PaddingRight = UDim.new(0,26); CP.PaddingBottom = UDim.new(0,26)
    local CL = Instance.new("UIListLayout", Content)
    CL.Padding = UDim.new(0,14); CL.SortOrder = Enum.SortOrder.LayoutOrder
    S.Sidebar = Sidebar; S.Content = Content

    for _, t in ipairs(S.Tabs) do
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(1,0,0,52); btn.BackgroundColor3 = SIDE_BG
        btn.Text = ""; btn.AutoButtonColor = false; btn.Parent = SS; Corner(btn, 12)
        local bs = Stroke(btn, BORDER, 1)
        local ind = Instance.new("Frame")
        ind.Size = UDim2.new(0,0,1,-16); ind.Position = UDim2.fromOffset(0,8)
        ind.BackgroundColor3 = PURPLE; ind.BorderSizePixel = 0
        ind.Parent = btn; Corner(ind, 2)
        local icoW = Instance.new("Frame")
        icoW.Size = UDim2.fromOffset(32,32); icoW.Position = UDim2.fromOffset(14,10)
        icoW.BackgroundColor3 = CARD_BG
        icoW.Parent = btn; Corner(icoW, 9)
        local ico = Instance.new("TextLabel")
        ico.Size = UDim2.fromScale(1,1); ico.BackgroundTransparency = 1
        ico.Text = t.icon; ico.TextColor3 = TXT; ico.TextSize = 15
        ico.Font = Enum.Font.Gotham; ico.Parent = icoW
        local lbl = Lbl(btn, t.name, 13, UDim2.fromOffset(56,8), Enum.Font.GothamBold)
        lbl.Size = UDim2.new(1,-64,0,18)
        local sub = Lbl(btn, t.desc, 9, UDim2.fromOffset(56,26), Enum.Font.Gotham, TXT_DIM)
        sub.Size = UDim2.new(1,-64,0,14)
        HookHover(btn, SIDE_BG, Color3.fromRGB(35,35,50))
        btn.MouseButton1Down:Connect(function() Ripple(btn) end)
        btn.MouseButton1Click:Connect(function() SelectTab(t.name) end)
        S.TabBtns[t.name] = {btn=btn, lbl=lbl, sub=sub, indicator=ind, stroke=bs}
        local pg = Instance.new("Frame")
        pg.Size = UDim2.new(1,0,0,0); pg.AutomaticSize = Enum.AutomaticSize.Y
        pg.BackgroundTransparency = 1; pg.Visible = false; pg.Parent = Content
        local pgl = Instance.new("UIListLayout", pg)
        pgl.Padding = UDim.new(0,14); pgl.SortOrder = Enum.SortOrder.LayoutOrder
        S.Pages[t.name] = pg
    end

    SearchBox:GetPropertyChangedSignal("Text"):Connect(function() AplicarBusca(SearchBox.Text) end)

    Ham.MouseButton1Click:Connect(function()
        S.SidebarAberta = not S.SidebarAberta
        local d = S.SidebarAberta and 0 or -SW
        local dc = S.SidebarAberta and SW or 0
        Tw(Sidebar, {Position = UDim2.fromOffset(d,68)}, 0.35, Enum.EasingStyle.Quart)
        Tw(Content, {Position = UDim2.fromOffset(dc,68), Size = UDim2.new(1,-dc,1,-68)}, 0.35, Enum.EasingStyle.Quart)
    end)

    local fpsOverlay = Instance.new("TextLabel")
    fpsOverlay.Size = UDim2.fromOffset(90, 24)
    fpsOverlay.AnchorPoint = Vector2.new(1, 0)
    fpsOverlay.Position = UDim2.new(1, -8, 0, 8)
    fpsOverlay.BackgroundColor3 = CARD_BG
    fpsOverlay.BackgroundTransparency = 0.3
    fpsOverlay.TextColor3 = PURPLE_LT; fpsOverlay.TextSize = 12
    fpsOverlay.Font = Enum.Font.GothamBold; fpsOverlay.Text = "--- FPS"
    fpsOverlay.Visible = false; fpsOverlay.Parent = Gui
    Corner(fpsOverlay, 8); Stroke(fpsOverlay, PURPLE, 1)
    S.FPSLabel = fpsOverlay
end

-- ===================== BUILD TABS =====================
local function BuildPrincipal()
    local pg = S.Pages["Principal"]
    do
        local c = Instance.new("Frame")
        c.Size = UDim2.new(1,0,0,130); c.BackgroundColor3 = CARD_BG
        c.Parent = pg
        Corner(c, 14); Stroke(c, BORDER, 1)
        Gradient(c, CARD_BG, BG, 90)
        local aw = Instance.new("Frame")
        aw.Size = UDim2.fromOffset(88,88); aw.Position = UDim2.fromOffset(22,21)
        aw.BackgroundColor3 = BG; aw.Parent = c
        Corner(aw, 44)
        local as = Stroke(aw, PURPLE, 2); AddRGB(as); Glow(aw, PURPLE, 0.6)
        local av = Instance.new("ImageLabel")
        av.Size = UDim2.new(1,-6,1,-6); av.Position = UDim2.fromOffset(3,3)
        av.BackgroundTransparency = 1; av.Image = ""; av.Parent = aw; Corner(av, 42)
        task.spawn(function()
            local ok, img = pcall(function()
                return Players:GetUserThumbnailAsync(LocalPlayer.UserId, Enum.ThumbnailType.HeadShot, Enum.ThumbnailSize.Size150x150)
            end)
            if ok and img then av.Image = img end
        end)
        Lbl(c, LocalPlayer.DisplayName, 20, UDim2.fromOffset(128,34), Enum.Font.GothamBlack, TXT).Size = UDim2.new(1,-140,0,26)
        Lbl(c, "@"..LocalPlayer.Name, 12, UDim2.fromOffset(128,62), Enum.Font.GothamMedium, TXT).Size = UDim2.new(1,-140,0,16)
        Lbl(c, "ID: "..tostring(LocalPlayer.UserId), 10, UDim2.fromOffset(128,82), Enum.Font.Gotham, TXT_DIM).Size = UDim2.new(1,-140,0,14)
        local bd = Instance.new("TextLabel")
        bd.Size = UDim2.fromOffset(80,24); bd.Position = UDim2.new(1,-96,0,22)
        bd.BackgroundColor3 = Color3.fromRGB(20,50,35)
        bd.Text = "SAFE"; bd.TextColor3 = GREEN; bd.TextSize = 10
        bd.Font = Enum.Font.GothamBold; bd.Parent = c
        Corner(bd, 12); Stroke(bd, GREEN, 1)
    end
    local cP2 = Card(pg, "DASHBOARD", BLUE)
    local function StatRow(par, tit, val, cor)
        local rf = Instance.new("Frame")
        rf.Size = UDim2.new(1,0,0,30); rf.BackgroundTransparency = 1; rf.Parent = par
        local d = Instance.new("Frame")
        d.Size = UDim2.fromOffset(6,6); d.Position = UDim2.fromOffset(0,12)
        d.BackgroundColor3 = cor; d.BorderSizePixel = 0
        d.Parent = rf; Corner(d, 3)
        Lbl(rf, tit, 12, UDim2.fromOffset(14,4), Enum.Font.GothamMedium).Size = UDim2.new(0.6,0,0,22)
        local v = Lbl(rf, val, 13, UDim2.new(0.6,0,0,0), Enum.Font.GothamBold, cor)
        v.Size = UDim2.new(0.4,0,0,22); v.TextXAlignment = Enum.TextXAlignment.Right
        return v
    end
    -- BUG FIX #13: contador com cleanup automático
    local function Contador(lbl, get, dur)
        task.spawn(function()
            local at = 0
            while Gui and Gui.Parent and lbl and lbl.Parent do
                local alvo = get()
                for i = 1, 12 do
                    if not lbl.Parent then return end
                    at = at + (alvo-at)*0.3
                    lbl.Text = tostring(math.floor(at + 0.5))
                    task.wait((dur or 0.4)/12)
                end
                if not lbl.Parent then return end
                lbl.Text = tostring(alvo)
                task.wait(0.5)
            end
        end)
    end
    local pV1 = StatRow(cP2, "Jogadores", tostring(#Players:GetPlayers()), BLUE)
    Contador(pV1, function() return #Players:GetPlayers() end, 0.4)
    local pV2 = StatRow(cP2, "Amigos", "0", GREEN)
    Contador(pV2, function() local c=0; for _ in pairs(Amigos) do c=c+1 end; return c end, 0.3)
    S.pV3 = StatRow(cP2, "WalkSpeed", tostring(CUR_SPEED), PURPLE_LT)
    local pV4 = StatRow(cP2, "Aimbot", "OFF", TXT_DIM)
    task.spawn(function()
        while Gui and Gui.Parent do
            task.wait(0.3)
            pcall(function()
                pV4.Text = Settings.Aimbot.Enabled and "ATIVO" or "OFF"
                pV4.TextColor3 = Settings.Aimbot.Enabled and GREEN or TXT_DIM
            end)
        end
    end)
    local pV5 = StatRow(cP2, "ESP", "OFF", TXT_DIM)
    task.spawn(function()
        while Gui and Gui.Parent do
            task.wait(0.3)
            pcall(function()
                pV5.Text = Settings.ESP.Enabled and "ATIVO" or "OFF"
                pV5.TextColor3 = Settings.ESP.Enabled and GREEN or TXT_DIM
            end)
        end
    end)
    local cWarn = Card(pg, "✅ FIXED v6.6.0", GREEN)
    local aviso = Lbl(cWarn, "Correções aplicadas:\n• GetVehicles com cache (perf)\n• AntiLag sem GetDescendants full\n• Sky restore corrigido\n• Neck R6 corrigido\n• WS restore no respawn\n• Leaks de ESP/Contador removidos", 11, UDim2.new(), Enum.Font.Gotham, TXT_DIM)
    aviso.Size = UDim2.new(1,0,0,90); aviso.TextWrapped = true
    local aviso2 = Lbl(cWarn, "🟢 Cliente (funciona): ESP, Chams, Aimbot, Trigger, NoRecoil, FOV, Fly, Noclip, Speed\n🟠 Server-side (limitado): AutoRevive, Trunk, Trocar Emprego — bloqueados por design do servidor", 10, UDim2.new(), Enum.Font.Gotham, TXT_DIM)
    aviso2.Size = UDim2.new(1,0,0,60); aviso2.TextWrapped = true
    local cAt = Card(pg, "ATIVIDADES EM TEMPO REAL", PURPLE_LT)
    AtividadeFrame = Instance.new("Frame", cAt)
    AtividadeFrame.Size = UDim2.new(1,0,0,240); AtividadeFrame.BackgroundTransparency = 1
    for i = #Atividades, 1, -1 do local a = Atividades[i]; AddAtividade(a.txt, a.cor) end
    AddAtividade("Painel v6.6.0 FIXED carregado", GREEN)
    AddAtividade("Proteção anti-kick ativa", GREEN)
end

local function BuildCombate()
    local pg = S.Pages["Combate"]
    Section(pg, "Combate", "Aimbot, Aimlock e Triggerbot")
    local c1 = Card(pg, "AIMBOT", RED)
    Toggle(c1, "Ativar Aimbot", "mira automatica", Settings.Aimbot, "Enabled")
    Toggle(c1, "Ignorar Time", "nao mira aliados", Config, "IgnorarTime")
    Toggle(c1, "Sempre Ativo", "segurar botao pra ativar", Config, "SempreAtivo")
    Dropdown(c1, "Botao Ativacao", {"Esquerdo","Direito"}, Config, "BotaoAtivacao")
    Toggle(c1, "Sem Arma", "mira sem arma", Config, "SemArma")
    Toggle(c1, "Visible Check", "so mira visivel", Settings.Aimbot, "WallCheck")
    Toggle(c1, "Predição", "compensa ping (0.1s)", Config, "Predicao")
    Toggle(c1, "FOV Circle", "circulo da mira", Config, "MostrarFOVCircle")
    Slider(c1, "FOV", 50, 600, Settings.Aimbot, "FOV")
    Slider(c1, "Distancia", 0, 5000, Config, "DistanciaAimbot")
    Slider(c1, "Suavidade", 1, 20, Settings.Aimbot, "Smoothness")
    Dropdown(c1, "Parte Alvo", {"Head","Neck","Chest"}, Settings.Aimbot, "Part")
    local cA = Card(pg, "AIMLOCK", PURPLE)
    Toggle(cA, "Ativar Aimlock", "trava suave no alvo", Settings.Aimlock, "Enabled")
    Slider(cA, "Velocidade do Puxao", 1, 30, Settings.Aimlock, "Velocidade")
    Slider(cA, "Forca", 0.1, 3.0, Settings.Aimlock, "Forca")
    Slider(cA, "Precisao", 1, 20, Settings.Aimlock, "Precisao")
    local c2 = Card(pg, "TRIGGERBOT", YELLOW)
    Toggle(c2, "Ativar Triggerbot", "atira automatico", Settings.Triggerbot, "Enabled")
    Dropdown(c2, "Modo", {"Circle","Crosshair"}, Settings.Triggerbot, "Mode")
    Slider(c2, "Raio", 20, 400, Settings.Triggerbot, "Radius")
    Slider(c2, "Delay", 0.01, 0.5, Settings.Triggerbot, "Delay")
    local c3 = Card(pg, "SENSITIVIDADE", GREEN)
    Toggle(c3, "Reduzir Sens.", "reduz MouseDeltaSensitivity", Settings.NoRecoil, "Enabled")
    Slider(c3, "Intensidade", 0, 1, Settings.NoRecoil, "Intensity")
end

local function BuildVisual()
    local pg = S.Pages["Visual"]
    Section(pg, "Visual", "ESP e Chams (client-side, seguro)")
    local c1 = Card(pg, "ESP · JOGADORES", BLUE)
    Toggle(c1, "Ativar ESP", "destaca inimigos", Settings.ESP, "Enabled")
    Toggle(c1, "Box ESP", "caixa ao redor", Settings.ESP, "Box")
    Dropdown(c1, "Estilo da Box", {"2D","3D","Corner"}, Settings.ESP, "BoxStyle")
    Toggle(c1, "Skeleton", "exibe ossos", Settings.ESP, "Skeleton")
    Toggle(c1, "Head Dot", "ponto na cabeca", Settings.ESP, "HeadDot")
    Toggle(c1, "Chams", "ver atraves de paredes", Settings.Chams, "Enabled")
    Toggle(c1, "Nomes", "nome e distancia", Settings.ESP, "ShowName")
    Toggle(c1, "Vida", "barra de vida", Settings.ESP, "Health")
    Toggle(c1, "Tool ESP", "arma equipada", Settings.ESP, "Weapon")
    Toggle(c1, "Ver Mochila", "itens da mochila", Settings.ESP, "ShowBackpack")
    Toggle(c1, "Linha Base", "linha base->pes", Settings.ESP, "Line")
    Toggle(c1, "Tracers", "linha camera->cabeca", Settings.ESP, "Tracers")
    Toggle(c1, "Time", "time do inimigo", Settings.ESP, "ShowTeam")
    Toggle(c1, "Team Check", "ignora mesmo time", Settings.ESP, "TeamCheck")
    Toggle(c1, "Cor por Time", "verde=aliado vermelho=inimigo", Settings.ESP, "ColorByTeam")
    Toggle(c1, "RGB", "cor ciclando", Settings.ESP, "RGB")
    Slider(c1, "Distancia ESP", 100, 10000, Settings.ESP, "Distance")
    Dropdown(c1, "Nome", {"Display","UserName"}, Settings.ESP, "NameMode")
    ColorRow(c1, "Cor do ESP", Settings.ESP, "ESPColor")
    ColorRow(c1, "Cor do Chams", Settings.Chams, "ChamsColor")
    ColorRow(c1, "Cor Visivel", Settings.ESP, "VisibleColor")
    ColorRow(c1, "Cor Parede", Settings.ESP, "WallColor")
end

local function BuildMovimento()
    local pg = S.Pages["Movimento"]
    Section(pg, "Movimento", "velocidade e extras")
    local cM1 = Card(pg, "SPEED HUB (LIMITADO)", ORANGE)
    local tr = Instance.new("Frame", cM1)
    tr.Size = UDim2.new(1,0,0,44); tr.BackgroundTransparency = 1
    Lbl(tr, "Ativar Speed Hub", 13, UDim2.fromOffset(0,12), Enum.Font.GothamMedium).Size = UDim2.new(1,-80,0,18)
    local ssw = Instance.new("TextButton", tr)
    ssw.Size = UDim2.fromOffset(46,26); ssw.Position = UDim2.new(1,-46,0.5,-13)
    ssw.BackgroundColor3 = SWITCH_OFF; ssw.Text = ""; ssw.AutoButtonColor = false
    Corner(ssw, 13)
    local sswS = Stroke(ssw, BORDER, 1)
    local sk = Instance.new("Frame", ssw)
    sk.Size = UDim2.fromOffset(20,20); sk.Position = UDim2.fromOffset(3,3)
    sk.BackgroundColor3 = Color3.new(1,1,1); sk.BorderSizePixel = 0; Corner(sk, 10)
    ssw.MouseButton1Click:Connect(function()
        SpeedAtivo = not SpeedAtivo
        if SpeedAtivo then
            Tw(ssw, {BackgroundColor3 = ORANGE}); Tw(sswS, {Color = ORANGE})
            Tw(sk, {Position = UDim2.fromOffset(23,3)}, 0.2, Enum.EasingStyle.Back)
            local h = GetHumanoid()
            if h then h.WalkSpeed = CUR_SPEED end
            Notify("Speed Hub: "..CUR_SPEED.." (max "..SAFE_SPEED_LIMIT..")", ORANGE, "⚠️")
        else
            Tw(ssw, {BackgroundColor3 = SWITCH_OFF}); Tw(sswS, {Color = BORDER})
            Tw(sk, {Position = UDim2.fromOffset(3,3)}, 0.2, Enum.EasingStyle.Back)
            local h = GetHumanoid()
            if h then h.WalkSpeed = BASE_SPEED end
            Notify("Speed Hub desativado (base: "..BASE_SPEED..")", RED, "X")
        end
    end)
    local sr = Instance.new("Frame", cM1)
    sr.Size = UDim2.new(1,0,0,34); sr.BackgroundTransparency = 1
    Lbl(sr, "Velocidade atual (máx. "..SAFE_SPEED_LIMIT..")", 13, UDim2.fromOffset(0,7), Enum.Font.GothamMedium).Size = UDim2.new(0.6,0,1,0)
    SpeedValueBox = Instance.new("TextBox", sr)
    SpeedValueBox.Size = UDim2.fromOffset(72,30); SpeedValueBox.Position = UDim2.new(1,-72,0.5,-15)
    SpeedValueBox.BackgroundColor3 = INPUT_BG
    SpeedValueBox.Text = tostring(CUR_SPEED); SpeedValueBox.TextColor3 = ORANGE
    SpeedValueBox.TextSize = 14; SpeedValueBox.Font = Enum.Font.GothamBold
    SpeedValueBox.ClearTextOnFocus = false
    Corner(SpeedValueBox, 8); Stroke(SpeedValueBox, ORANGE)
    SpeedValueBox.FocusLost:Connect(function()
        local n = tonumber(SpeedValueBox.Text)
        if n then SetSpeed(n) else SpeedValueBox.Text = tostring(CUR_SPEED) end
    end)
    local cM2 = Card(pg, "AVANCADO", YELLOW)
    Toggle(cM2, "WalkSpeed Custom", "custom (risco alto)", Settings.WalkSpeed, "Enabled", nil, true)
    Slider(cM2, "Velocidade Custom", 16, 79, Settings.WalkSpeed, "Value")
    Toggle(cM2, "Underground", "atravessa subsolo (15s)", Settings.Underground, "Enabled", nil, true)
    Slider(cM2, "Profundidade", 1, 20, Settings.Underground, "Depth")
    Toggle(cM2, "Spinbot", "gira personagem", Settings.Spinbot, "Enabled")
    Slider(cM2, "Vel. Spin", 60, 9000, Settings.Spinbot, "Speed")
    Toggle(cM2, "Auto Jump", "pula automatico", Settings.AutoJump, "Enabled")
    Toggle(cM2, "Arma Gigante", "aumenta arma (60s)", Settings.GiantGun, "Enabled", nil, true)
end

local function BuildJogador()
    local pg = S.Pages["Jogador"]
    Section(pg, "Jogador", "amigos e colar")
    local c1 = Card(pg, "AMIGOS", GREEN)
    local cntL = Lbl(c1, "Amigos: 0", 12, UDim2.new(), Enum.Font.GothamBold, PURPLE_LT)
    cntL.Size = UDim2.new(1,0,0,20)
    local search = Instance.new("TextBox", c1)
    search.Size = UDim2.new(1,0,0,36); search.BackgroundColor3 = INPUT_BG
    search.PlaceholderText = "🔍 Buscar jogador..."
    search.PlaceholderColor3 = TXT_DIM; search.Text = ""
    search.TextColor3 = TXT; search.TextSize = 12; search.Font = Enum.Font.Gotham
    search.ClearTextOnFocus = false; Corner(search, 10); Stroke(search, BORDER)
    BigBtn(c1, "Adicionar todos", function()
        for _, p in ipairs(Players:GetPlayers()) do if p ~= LocalPlayer then AddAmigo(p.Name) end end
    end)
    BigBtn(c1, "Limpar todos", function() LimparAmigos() end, true)
    local lf = Instance.new("Frame", c1)
    lf.Size = UDim2.new(1,0,0,180); lf.BackgroundColor3 = INPUT_BG
    Corner(lf, 12); Stroke(lf, BORDER)
    local ls = Instance.new("ScrollingFrame", lf)
    ls.Size = UDim2.new(1,-10,1,-10); ls.Position = UDim2.fromOffset(5,5)
    ls.BackgroundTransparency = 1; ls.BorderSizePixel = 0
    ls.ScrollBarThickness = 2; ls.ScrollBarImageColor3 = PURPLE
    ls.AutomaticCanvasSize = Enum.AutomaticSize.Y; ls.CanvasSize = UDim2.new()
    local ll = Instance.new("UIListLayout", ls)
    ll.Padding = UDim.new(0,4); ll.SortOrder = Enum.SortOrder.Name
    local function Refresh()
        local q = search.Text:lower()
        for _, c in ipairs(ls:GetChildren()) do if c:IsA("Frame") then c:Destroy() end end
        local c = 0; for _ in pairs(Amigos) do c = c + 1 end
        cntL.Text = "Amigos: "..c
        for _, p in ipairs(Players:GetPlayers()) do
            if p ~= LocalPlayer and (q == "" or p.Name:lower():find(q)) then
                local isF = Amigos[p.Name] == true
                local it = Instance.new("Frame", ls)
                it.Size = UDim2.new(1,-4,0,34)
                it.BackgroundColor3 = isF and Color3.fromRGB(60,25,90) or CARD_BG
                Corner(it, 10)
                Lbl(it, p.Name, 12, UDim2.fromOffset(14,8), Enum.Font.GothamBold).Size = UDim2.new(0.6,0,0,18)
                local fb = Instance.new("TextButton", it)
                fb.Size = UDim2.fromOffset(90,24); fb.Position = UDim2.new(1,-96,0.5,-12)
                fb.BackgroundColor3 = isF and Color3.fromRGB(80,25,35) or Color3.fromRGB(50,30,80)
                fb.Text = isF and "REMOVER" or "ADICIONAR"
                fb.TextColor3 = TXT; fb.TextSize = 9
                fb.Font = Enum.Font.GothamBold; fb.AutoButtonColor = false
                Corner(fb, 12)
                fb.MouseButton1Click:Connect(function()
                    if Amigos[p.Name] then RemAmigo(p.Name) else AddAmigo(p.Name) end
                    Refresh()
                end)
            end
        end
    end
    search:GetPropertyChangedSignal("Text"):Connect(Refresh)
    Players.PlayerAdded:Connect(function() task.wait(0.3); Refresh() end)
    Players.PlayerRemoving:Connect(function() task.wait(0.1); Refresh() end)
    task.spawn(Refresh)
    local cColar = Card(pg, "COLAR (RISCO)", ORANGE)
    Toggle(cColar, "Ativar Colar", "cola no alvo (20s)", Settings.Colar, "Enabled", nil, true)
    local aviso = Lbl(cColar, "⚠️ Auto-desliga após 20 segundos", 10, UDim2.new(), Enum.Font.Gotham, TXT_DIM)
    aviso.Size = UDim2.new(1,0,0,20)
end

local function BuildUtil()
    local pg = S.Pages["Util"]
    Section(pg, "Util", "noclip e visao")
    local c1 = Card(pg, "NOCLIP", ORANGE)
    Toggle(c1, "Ativar Noclip", "atravessa paredes (30s)", Config, "Noclip", function(en)
        if not en then NoclipTimer = 0 end
        AplicarNoclip(en)
    end, true)
    InputRow(c1, "Tecla Atalho", Config, "NoclipKey", function(t)
        local s = t:upper()
        if #s == 1 and (s:match("%a") or s:match("%d")) then
            if SafeKeyCode(s) then return true, s end
        end
        return false
    end)
    local c2 = Card(pg, "VISAO", BLUE)
    Toggle(c2, "Ativar FOV Camera", "aplica FOV customizado", Config, "FOVAtivo", function(en)
        if not en then Camera.FieldOfView = 70; Notify("FOV desativado", RED, "X")
        else Camera.FieldOfView = Config.FOV_Camera; Notify("FOV ativado: "..Config.FOV_Camera, GREEN, "✓") end
    end)
    Slider(c2, "FOV Camera", 0, 240, Config, "FOV_Camera")
    local c3 = Card(pg, "EXTRAS (RISCO)", ORANGE)
    Toggle(c3, "God Mode", "vida infinita (detectável)", Settings.GodMode, "Enabled", nil, true)
    Toggle(c3, "Auto Reviver", "revive local (server pode reverter)", Settings.AutoRevive, "Enabled")
end

local function BuildMundo()
    local pg = S.Pages["Mundo"]
    Section(pg, "Mundo", "iluminação e ambiente")
    local c1 = Card(pg, "ILUMINAÇÃO", YELLOW)
    Toggle(c1, "Alterar Horário", "trava o horário do dia", Settings.World, "TimeEnabled")
    Slider(c1, "Horário", 0, 24, Settings.World, "TimeValue")
    Toggle(c1, "Alterar Brilho", "trava o brilho do ambiente", Settings.World, "BrightnessEnabled")
    Slider(c1, "Brilho", 0, 10, Settings.World, "BrightnessValue")
    Toggle(c1, "Remover Névoa", "remove neblina e atmosfera", Settings.World, "RemoveFog")
    Toggle(c1, "Fullbright", "iluminação máxima sem sombras", Settings.World, "Fullbright")
end

local function BuildDesempenho()
    local pg = S.Pages["Desempenho"]
    Section(pg, "Desempenho", "FPS e otimização (raio 500 studs)")
    local c1 = Card(pg, "FPS", BLUE)
    Slider(c1, "Limite de FPS", 30, 970, Settings.Performance, "FPSLimit", AplicarFPSLimit)
    Toggle(c1, "Exibir FPS", "FPS no canto da tela", Settings.Performance, "ShowFPS", function(en)
        if S.FPSLabel then S.FPSLabel.Visible = en end
    end)
    local c2 = Card(pg, "OTIMIZAÇÃO", GREEN)
    Toggle(c2, "Remover Texturas", "só no raio de 500 studs", Settings.Performance, "RemoveTextures", function() AplicarOtimizacao() end)
    Toggle(c2, "Anti Lag", "só no raio de 500 studs", Settings.Performance, "AntiLag", function() AplicarOtimizacao() end)
    Toggle(c2, "Remover Sombras", "só no raio de 500 studs", Settings.Performance, "RemoveShadows", function() AplicarOtimizacao() end)
    Toggle(c2, "Anti AFK", "movimenta o mouse a cada 15 min", Settings.Performance, "AntiAFK")
end

local function BuildVeiculos()
    local pg = S.Pages["Veículos"]
    Section(pg, "Veículos", "ESP, controle e utilidades")
    local cEsp = Card(pg, "VEHICLE ESP", BLUE)
    Toggle(cEsp, "Vehicle ESP", "destaca veículos próximos", Settings.Vehicle, "ESPEnabled")
    Slider(cEsp, "Distância Máxima", 100, 5000, Settings.Vehicle, "ESPDistance")
    ColorRow(cEsp, "Cor do ESP", Settings.Vehicle, "ESPColor")
    local cUtil = Card(pg, "UTILIDADES", GREEN)
    Toggle(cUtil, "NoClip do Veículo", "remove colisão", Settings.Vehicle, "NoClip")
    Toggle(cUtil, "Combustível Infinito", "tanque sempre cheio", Settings.Vehicle, "InfiniteFuel")
    Toggle(cUtil, "Vida do Veículo", "integridade máxima", Settings.Vehicle, "Invincible")
    Toggle(cUtil, "Anti PIT", "impede travamento policial", Settings.Vehicle, "AntiPIT")
    Toggle(cUtil, "Engine Boost", "melhora motor", Settings.Vehicle, "EngineBoost")
    local cMotor = Card(pg, "MOTOR", PURPLE_LT)
    Slider(cMotor, "Potência (HP)", 100, 100000, Settings.Vehicle, "Horsepower")
    local cTrunk = Card(pg, "PORTA-MALAS", YELLOW)
    local trRow = Instance.new("Frame", cTrunk)
    trRow.Size = UDim2.new(1,0,0,46); trRow.BackgroundTransparency = 1
    Lbl(trRow, "Itens no Porta-Malas", 13, UDim2.fromOffset(0,14), Enum.Font.GothamMedium).Size = UDim2.new(0.6,0,0,18)
    local trLbl = Lbl(trRow, "None", 12, UDim2.new(0.6,0,0,0), Enum.Font.GothamBold, TXT_DIM)
    trLbl.Size = UDim2.new(0.4,-10,0,46); trLbl.TextXAlignment = Enum.TextXAlignment.Right
    S.TrunkItemsLabel = trLbl
    BigBtn(cTrunk, "Atualizar Itens", function() RefreshTrunkItems() end)
    Lbl(cTrunk, "⚠️ Abrir/guardar porta-malas é server-side (dossiê push-only) — não implementado", 10, UDim2.new(), Enum.Font.Gotham, ORANGE).Size = UDim2.new(1,0,0,32)
    local cMov = Card(pg, "MOVIMENTO", PURPLE)
    BigBtn(cMov, "Entrar no Veículo", function() EntrarVeiculo(Settings.Vehicle.SelectedVehicle) end)
    local selRow = Instance.new("Frame", cMov)
    selRow.Size = UDim2.new(1,0,0,46); selRow.BackgroundTransparency = 1
    Lbl(selRow, "Veículo selecionado", 13, UDim2.fromOffset(0,14), Enum.Font.GothamMedium).Size = UDim2.new(0.6,0,0,18)
    local vehLbl = Lbl(selRow, "None", 12, UDim2.new(0.6,0,0,0), Enum.Font.GothamBold, PURPLE_LT)
    vehLbl.Size = UDim2.new(0.4,-10,0,46); vehLbl.TextXAlignment = Enum.TextXAlignment.Right
    S.VehicleListLabel = vehLbl
    BigBtn(cMov, "Selecionar Veículo (mais próximo)", function()
        local vehs = GetVehicles(true)
        local mr = GetHRP()
        if not mr or #vehs == 0 then Notify("Nenhum veículo", RED, "X"); return end
        local closest, cd = nil, math.huge
        for _, v in ipairs(vehs) do
            local root = v.PrimaryPart or v:FindFirstChildWhichIsA("BasePart")
            if root then
                local d = (root.Position - mr.Position).Magnitude
                if d < cd then cd = d; closest = v end
            end
        end
        if closest then
            Settings.Vehicle.SelectedVehicle = closest
            vehLbl.Text = GetVehicleName(closest).." ("..math.floor(cd).."m)"
            Notify("Selecionado: "..GetVehicleName(closest), GREEN, "✓")
        end
    end)
    Toggle(cMov, "Voo do Veículo", "WASD/Space/Ctrl", Settings.Vehicle, "Fly")
    Toggle(cMov, "Fling", "arremessa contra o selecionado", Settings.Vehicle, "Fling")
    Slider(cMov, "Velocidade Voo", 100, 2000, Settings.Vehicle, "FlySpeed")
    Slider(cMov, "Boost Multiplier", 1, 20, Settings.Vehicle, "FlyBoostMult")
    Slider(cMov, "Aceleração do Voo", 1, 50, Settings.Vehicle, "FlyAccel")
end

local function BuildConfig()
    local pg = S.Pages["Config"]
    Section(pg, "Config", "configurações e emprego")
    local cUI = Card(pg, "INTERFACE", PURPLE)
    Dropdown(cUI, "Tema", {"Padrão","Escuro","Claro","Neon"}, Settings.Config, "Theme", function(v)
        Notify("Tema: "..v, PURPLE_LT, "🎨")
    end)
    InputRow(cUI, "Atalho do Menu", Settings.Config, "MenuKey", function(t)
        local s = t:upper()
        if SafeKeyCode(s) then return true, s end
        return false
    end)
    BigBtn(cUI, "Encerrar Kelp", function() Gui:Destroy() end, true)
    local cJob = Card(pg, "EMPREGOS (SERVER-SIDE)", ORANGE)
    Lbl(cJob, "⚠️ Trocar emprego requer RemoteFunction com schema desconhecido (dossiê push-only). Não implementado.", 11, UDim2.new(), Enum.Font.Gotham, ORANGE).Size = UDim2.new(1,0,0,60)
    Dropdown(cJob, "Selecionar Emprego (cosmético)", {"PM","PF","PRF","ROTA","BOPE","GCM","PC","Civil","Mecânico","Médico","Taxista","Entregador"}, Settings.Config, "SelectedJob")
    BigBtn(cJob, "Trocar de Emprego", function() TrocarEmprego(Settings.Config.SelectedJob) end)
    local c1 = Card(pg, "GERENCIAMENTO", BLUE)
    local function Confete()
        for i = 1, 40 do
            local p = Instance.new("Frame")
            p.Size = UDim2.fromOffset(math.random(4,8), math.random(4,8))
            p.Position = UDim2.new(0.5,0,0.5,0)
            p.BackgroundColor3 = Color3.fromHSV(math.random(),0.8,1)
            p.BorderSizePixel = 0; p.ZIndex = 100; p.Parent = Gui
            Corner(p, 2)
            local ang = math.random()*math.pi*2; local dist = math.random(80,260)
            Tw(p, {Position = UDim2.new(0.5, math.cos(ang)*dist, 0.5, math.sin(ang)*dist), Rotation = math.random(-360,360), BackgroundTransparency = 1}, 1.4, Enum.EasingStyle.Quad)
            task.delay(1.4, function() if p and p.Parent then p:Destroy() end end)
        end
    end
    BigBtn(c1, "💾  Salvar Config", function()
        local data = { Settings = Settings, Config = Config }
        local ok = pcall(function() writefile("TropaDolkk.json", HttpService:JSONEncode(data)) end)
        if ok then Notify("Configurações salvas!", GREEN, "✓"); Confete()
        else Notify("Executor sem suporte a writefile", RED, "X") end
    end)
    BigBtn(c1, "📂  Carregar Config", function()
        local ok = pcall(function()
            if isfile and isfile("TropaDolkk.json") then
                local r = HttpService:JSONDecode(readfile("TropaDolkk.json"))
                if r.Settings then for k, v in pairs(r.Settings) do if type(Settings[k]) == type(v) then Settings[k] = v end end end
                if r.Config then for k, v in pairs(r.Config) do if type(Config[k]) == type(v) then Config[k] = v end end end
                Notify("Configurações carregadas!", GREEN, "✓")
            end
        end)
        if not ok then Notify("Erro ao carregar", RED, "X") end
    end, true)
    BigBtn(c1, "🔄  Resetar Painel", function()
        S.Panel.Size = UDim2.fromOffset(1000, 600)
        S.Panel.Position = UDim2.fromScale(0.5, 0.5)
        Notify("Painel resetado", PURPLE_LT, "🔄")
    end)
    local c2 = Card(pg, "LOAD-SAFE", GREEN)
    Toggle(c2, "Stealth Mode", "força limites anti-kick", Config, "StealthMode")
    Lbl(c2, "• Speed Hub limitado a "..SAFE_SPEED_LIMIT, 10, UDim2.new(), Enum.Font.Gotham, TXT_DIM).Size = UDim2.new(1,0,0,16)
    Lbl(c2, "• Noclip auto-desliga em "..AutoDisableTimers.noclip.."s", 10, UDim2.new(), Enum.Font.Gotham, TXT_DIM).Size = UDim2.new(1,0,0,16)
    Lbl(c2, "• Underground auto-desliga em "..AutoDisableTimers.underground.."s", 10, UDim2.new(), Enum.Font.Gotham, TXT_DIM).Size = UDim2.new(1,0,0,16)
    Lbl(c2, "• Colar auto-desliga em "..AutoDisableTimers.colar.."s", 10, UDim2.new(), Enum.Font.Gotham, TXT_DIM).Size = UDim2.new(1,0,0,16)
    local c3 = Card(pg, "SOBRE", PURPLE_LT)
    Lbl(c3, "Tropa do LKK — v6.6.0 (FIXED)", 11, UDim2.new(), Enum.Font.Gotham, TXT_DIM).Size = UDim2.new(1,0,0,20)
    Lbl(c3, "Kelp Fusion · Caching · Nil-safe · Enum-safe", 10, UDim2.new(), Enum.Font.Gotham, GREEN).Size = UDim2.new(1,0,0,16)
end

-- ===================== BUILD TUDO =====================
BuildLogin()
BuildPanel()
BuildPrincipal()
BuildCombate()
BuildVisual()
BuildMovimento()
BuildJogador()
BuildUtil()
BuildMundo()
BuildDesempenho()
BuildVeiculos()
BuildConfig()

-- ===================== DRAG + RESIZE =====================
do
    local Panel = S.Panel
    local Header = S.Header
    local MIN_W, MIN_H = 700, 450
    local MAX_W, MAX_H = 1400, 900
    local dragging = false
    local dragStart, startPos = nil, nil
    Header.InputBegan:Connect(function(input)
        if input.UserInputType ~= Enum.UserInputType.MouseButton1 then return end
        local mpos = UserInputService:GetMouseLocation()
        local guis = PlayerGui:GetGuiObjectsAtPosition(mpos.X, mpos.Y)
        for _, g in ipairs(guis) do
            if g ~= Header and (g:IsA("TextButton") or g:IsA("ImageButton") or g:IsA("TextBox")) and g:IsDescendantOf(Header) then return end
        end
        dragging = true; dragStart = mpos; startPos = Panel.Position
    end)
    UserInputService.InputChanged:Connect(function(input)
        if not dragging then return end
        if input.UserInputType ~= Enum.UserInputType.MouseMovement then return end
        local cur = UserInputService:GetMouseLocation()
        local delta = cur - dragStart
        Panel.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end)
    local handle = Instance.new("Frame")
    handle.Size = UDim2.fromOffset(36, 36)
    handle.Position = UDim2.new(1, -42, 1, -42)
    handle.BackgroundTransparency = 1
    handle.ZIndex = 30; handle.Parent = Panel
    local handleBtn = Instance.new("TextButton")
    handleBtn.Size = UDim2.fromScale(1,1); handleBtn.BackgroundTransparency = 1
    handleBtn.Text = ""; handleBtn.ZIndex = 31; handleBtn.Parent = handle
    local linhas = {}
    for i = 1, 3 do
        local ln = Instance.new("Frame")
        ln.Size = UDim2.fromOffset(4 + i * 3, 2)
        ln.Position = UDim2.new(1, -10 - i * 4, 1, -10 - i * 4)
        ln.BackgroundColor3 = PURPLE_LT; ln.BackgroundTransparency = 1
        ln.BorderSizePixel = 0; ln.ZIndex = 32; ln.Parent = handle
        Corner(ln, 1); table.insert(linhas, ln)
    end
    handleBtn.MouseEnter:Connect(function()
        for _, ln in ipairs(linhas) do Tw(ln, {BackgroundColor3 = PURPLE, BackgroundTransparency = 0}, 0.2) end
    end)
    handleBtn.MouseLeave:Connect(function()
        for _, ln in ipairs(linhas) do Tw(ln, {BackgroundColor3 = PURPLE_LT, BackgroundTransparency = 1}, 0.25) end
    end)
    local resizing = false
    local startSize, startMouse = nil, nil
    handleBtn.InputBegan:Connect(function(input)
        if input.UserInputType ~= Enum.UserInputType.MouseButton1 then return end
        resizing = true; startSize = Panel.AbsoluteSize; startMouse = UserInputService:GetMouseLocation()
    end)
    UserInputService.InputChanged:Connect(function(input)
        if not resizing then return end
        if input.UserInputType ~= Enum.UserInputType.MouseMovement then return end
        local cur = UserInputService:GetMouseLocation()
        local dx = cur.X - startMouse.X; local dy = cur.Y - startMouse.Y
        local nw = math.clamp(startSize.X + dx, MIN_W, MAX_W)
        local nh = math.clamp(startSize.Y + dy, MIN_H, MAX_H)
        Panel.Size = UDim2.fromOffset(nw, nh)
    end)
    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false; resizing = false end
    end)
end

-- ===================== TECLAS =====================
UserInputService.InputBegan:Connect(function(input, gpe)
    if gpe then return end
    local noclipKC = SafeKeyCode(Config.NoclipKey)
    local menuKC   = SafeKeyCode(Settings.Config.MenuKey)
    local openKC   = SafeKeyCode(Config.OpenKey)

    if noclipKC and input.KeyCode == noclipKC then
        Config.Noclip = not Config.Noclip
        AplicarNoclip(Config.Noclip)
        Notify("Noclip: "..tostring(Config.Noclip), Config.Noclip and ORANGE or RED, Config.Noclip and "⚠️" or "X")
    end
    if (menuKC and input.KeyCode == menuKC) or (openKC and input.KeyCode == openKC) then
        if S.Panel then S.Panel.Visible = not S.Panel.Visible end
    end
    if K_UP and input.KeyCode == K_UP then SetSpeed(CUR_SPEED + 1) end
    if K_DOWN and input.KeyCode == K_DOWN then SetSpeed(CUR_SPEED - 1) end
    if K_HOME and input.KeyCode == K_HOME then SetSpeed(BASE_SPEED) end
    if K_F and K_LCTRL and input.KeyCode == K_F and UserInputService:IsKeyDown(K_LCTRL) then
        if S.Panel and S.Panel.Visible and S.SearchBox then S.SearchBox:CaptureFocus() end
    end
end)

-- ===================== LOGIN LOGIC =====================
local verificandoLogin = false
local function DoLogin()
    if verificandoLogin then return end
    local txt = S.PassInput.Text
    if txt == "" then S.ErrorLbl.Text = "Digite uma senha."; return end
    if txt == SENHA then
        verificandoLogin = true
        S.ErrorLbl.Text = ""; S.EnterBtn.Text = "Verificando"
        local dots = 0; local stop = false
        task.spawn(function()
            while not stop do
                task.wait(0.25); dots = (dots % 3) + 1
                if not stop then S.EnterBtn.Text = "Verificando"..string.rep(".", dots) end
            end
        end)
        task.wait(0.9)
        stop = true
        S.Login.Visible = false; S.Panel.Visible = true
        SelectTab("Principal")
        Notify("Painel v6.6.0 FIXED carregado!", GREEN, "✅")
    else
        S.ErrorLbl.Text = "Senha incorreta."; S.PassInput.Text = ""
        local orig = S.LoginCard.Position
        for i = 1, 6 do
            Tw(S.LoginCard, {Position = orig + UDim2.fromOffset((i % 2 == 0) and 12 or -12, 0)}, 0.05)
            task.wait(0.05)
        end
        Tw(S.LoginCard, {Position = orig}, 0.1)
    end
end
S.EnterBtn.MouseButton1Click:Connect(DoLogin)
S.PassInput.FocusLost:Connect(function(e) if e then DoLogin() end end)

Notify("Script v6.6.0 (FIXED) carregado! Senha: 2011", GREEN, "✅")
