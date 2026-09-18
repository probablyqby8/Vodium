--[[
    Vodium 1.0 — Universal Roblox Executor Script
    HVH-focused · Universal ESP · Modular · Bug-free
    All features off by default.
]]

-- ============================================================
-- SERVICES
-- ============================================================
local Players           = game:GetService("Players")
local RunService        = game:GetService("RunService")
local UserInputService  = game:GetService("UserInputService")
local TweenService      = game:GetService("TweenService")
local Lighting          = game:GetService("Lighting")
local SoundService      = game:GetService("SoundService")
local HttpService       = game:GetService("HttpService")
local TeleportService   = game:GetService("TeleportService")
local Workspace         = game:GetService("Workspace")

local LP     = Players.LocalPlayer
local Camera = Workspace.CurrentCamera

-- ============================================================
-- CONSTANTS
-- ============================================================
local SCRIPT_VERSION   = "1.0"
local CFG_PATH         = "vodium_v1_cfg.json"
local DEFAULT_HITSOUND = "rbxassetid://6894883033"
local DEFAULT_SKEET    = "rbxassetid://6042053626"

-- ============================================================
-- EXECUTOR COMPAT
-- ============================================================
local function safeCall(fn, ...)
    if type(fn) ~= "function" then return false end
    local ok, res = pcall(fn, ...)
    if ok then return true, res end
    return false, nil
end

local function getHui()
    local ok, h = safeCall(function() return gethui() end)
    if ok and h then return h end
    ok, h = safeCall(function() return game:GetService("CoreGui") end)
    if ok and h then return h end
    local pg = LP:FindFirstChildOfClass("PlayerGui")
    if pg then return pg end
    return LP:WaitForChild("PlayerGui", 10)
end

local function protectGui(g)
    safeCall(function() if protect_gui then protect_gui(g) end end)
    safeCall(function() if syn and syn.protect_gui then syn.protect_gui(g) end end)
end

local function setClipboard(str)
    safeCall(function() if setclipboard then setclipboard(str) end end)
    safeCall(function() if toclipboard then toclipboard(str) end end)
end

local function readFile(p)
    local ok, d = safeCall(function() return readfile(p) end)
    if ok then return d end
    return nil
end

local function writeFile(p, d)
    safeCall(function() if writefile then writefile(p, d) end end)
end

local function fileExists(p)
    local ok, r = safeCall(function() return isfile(p) end)
    return ok and r
end

local function setFpsCap(n)
    safeCall(function() if setfpscap then setfpscap(n) end end)
end

local function identifyExecutor()
    local ok, r = safeCall(function() return identifyexecutor() end)
    if ok and r then return r end
    return "Unknown"
end

-- ============================================================
-- STATE
-- ============================================================
local V = {
    Unloaded = false,
    Connections = {},
    Instances = {},
    Threads = {},
    ExtraCleanup = {},
}

local function Conn(sig, fn)
    local c = sig:Connect(fn)
    table.insert(V.Connections, c)
    return c
end

local function Track(i)
    table.insert(V.Instances, i)
    return i
end

local function OnUnload(fn)
    table.insert(V.ExtraCleanup, fn)
end

-- ============================================================
-- PALETTE
-- ============================================================
local PAL = {
    bg        = Color3.fromRGB(10,  10,  16),
    panel     = Color3.fromRGB(16,  16,  24),
    card      = Color3.fromRGB(22,  22,  32),
    cardHover = Color3.fromRGB(28,  28,  40),
    elevated  = Color3.fromRGB(30,  30,  44),
    stroke    = Color3.fromRGB(40,  40,  58),
    accent1   = Color3.fromRGB(139, 92,  246),
    accent2   = Color3.fromRGB(59,  130, 246),
    accent3   = Color3.fromRGB(236, 72,  153),
    ok        = Color3.fromRGB(16,  185, 129),
    err       = Color3.fromRGB(239, 68,  68),
    warn      = Color3.fromRGB(245, 158, 11),
    text      = Color3.fromRGB(226, 226, 235),
    muted     = Color3.fromRGB(113, 113, 122),
    enemy     = Color3.fromRGB(255, 70,  70),
    ally      = Color3.fromRGB(60,  220, 110),
    white     = Color3.fromRGB(255, 255, 255),
}

-- ============================================================
-- CONFIG
-- ============================================================
local Cfg = {
    -- ESP
    EspEnable      = false,
    EspTeamCheck   = true,
    EspMaxDist     = 1500,
    EspBox         = true,
    EspCorner      = false,
    EspFill        = false,
    EspHeadDot     = true,
    EspCornerLen   = 0.25,
    EspNames       = true,
    EspDisplayName = false,
    EspHealthBar   = true,
    EspHealthText  = false,
    EspDistance    = true,
    EspWeapon      = false,
    EspSkeleton    = false,
    EspSkelThick   = 1.5,
    EspTracers     = false,
    EspTracerOrigin= "Bottom",
    EspChams       = false,
    EspOffscreen   = false,
    EspTeamColor   = false,
    EspVisibleDot  = false,

    -- Aimbot
    AimEnable      = false,
    AimFOV         = 150,
    AimSmooth      = 0.5,
    AimMaxDist     = 1000,
    AimBone        = "Head",
    AimKey         = "MouseButton2",
    AimWallCheck   = true,
    AimTeamCheck   = true,
    AimPredict     = true,
    AimPredScale   = 1.0,
    AimBulletSpeed = 600,
    AimFovCircle   = false,

    -- Rage / HVH
    RageEnable     = false,
    RageAim        = false,
    RageFOV        = 360,
    RagePriority   = "Distance",
    RageAutoFire   = false,
    RageAutoReload = false,
    RageAutoScope  = false,
    RageWallPen    = true,
    AntiFling      = true,
    AntiVoid       = true,
    AutoRespawn    = false,

    -- Teleport
    TpDest         = nil,

    -- Hitsounds
    HitEnable      = false,
    HitStyle       = "Rust",
    HitVolume      = 1.5,
    HitHeadshot    = true,
    HitRage        = false,

    -- Crosshair
    ChEnable       = false,
    ChSize         = 10,
    ChThick        = 2,
    ChGap          = 4,
    ChColor        = {255, 255, 255},
    ChDot          = true,
    ChOutline      = true,

    -- Visuals
    Fullbright     = false,
    NoFog          = false,
    LowGfx         = false,
    CustomFOV      = false,
    FovValue       = 90,
    SpinVisual     = false,
    SpinSpeed      = 60,

    -- Radar
    RadarEnable    = false,
    RadarRadius    = 90,
    RadarScale     = 2.5,
    RadarOpacity   = 0.35,

    -- Player list
    PlistEnable    = false,

    -- Movement
    FlyEnable      = false,
    FlySpeed       = 60,
    SpeedHack      = false,
    WalkSpeed      = 16,
    InfJump        = false,
    Noclip         = false,
    AntiAfk        = true,

    -- Misc
    UncapFPS       = false,
    GuiScale       = 1.0,
    GuiVisible     = true,
}

for k, v in pairs(Cfg) do
    if type(v) == "boolean" then Cfg[k] = false end
end
Cfg.AntiAfk = true
Cfg.GuiScale = 1.0
Cfg.GuiVisible = true

local function SaveCfg()
    local t = {}
    for k, v in pairs(Cfg) do
        local vt = type(v)
        if vt == "boolean" or vt == "number" or vt == "string" then
            t[k] = v
        elseif vt == "table" then
            local simple = true
            for _, e in pairs(v) do
                if type(e) ~= "number" then simple = false break end
            end
            if simple then t[k] = v end
        end
    end
    local ok, enc = pcall(function() return HttpService:JSONEncode(t) end)
    if ok then writeFile(CFG_PATH, enc) end
end

local function LoadCfg()
    if not fileExists(CFG_PATH) then return end
    local raw = readFile(CFG_PATH)
    if not raw then return end
    local ok, t = pcall(function() return HttpService:JSONDecode(raw) end)
    if not ok or type(t) ~= "table" then return end
    for k, v in pairs(t) do
        if Cfg[k] ~= nil and type(Cfg[k]) == type(v) then
            Cfg[k] = v
        end
    end
end

-- ============================================================
-- UTILITY FUNCTIONS
-- ============================================================
local function New(cls, parent, props)
    local i = Instance.new(cls)
    for k, val in pairs(props or {}) do
        i[k] = val
    end
    if parent then i.Parent = parent end
    return i
end

local function AddCorner(parent, r)
    return New("UICorner", parent, { CornerRadius = UDim.new(0, r or 8) })
end

local function AddStroke(parent, color, thick, trans)
    return New("UIStroke", parent, {
        Color = color or PAL.stroke,
        Thickness = thick or 1,
        Transparency = trans or 0.2,
    })
end

local function Tween(inst, info, props)
    TweenService:Create(inst, info, props):Play()
end

local function ScreenCenter()
    if not Camera then return Vector2.new(0, 0) end
    local vp = Camera.ViewportSize
    return Vector2.new(vp.X * 0.5, vp.Y * 0.5)
end

local function IsKeyDown(name)
    local kc = Enum.KeyCode[name]
    if not kc then return false end
    return UserInputService:IsKeyDown(kc)
end

local function AimKeyHeld()
    local k = Cfg.AimKey
    if k == "Always" then return true end
    if k == "MouseButton2" then
        return UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2)
    elseif k == "MouseButton1" then
        return UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton1)
    end
    return IsKeyDown(k)
end

local function ClickMouse()
    safeCall(function() mouse1click() end)
    safeCall(function()
        local VU = game:GetService("VirtualUser")
        VU:Button1Down(Vector2.new(0, 0), Camera.CFrame)
        task.wait(0.02)
        VU:Button1Up(Vector2.new(0, 0), Camera.CFrame)
    end)
end

local function Notify(title, message, duration)
    if not V.Notify then return end
    V.Notify(title, message, duration)
end

-- ============================================================
-- UNIVERSAL CHARACTER ANALYZER
-- ============================================================
local R6_BONES = {
    {"Head", "Torso"},
    {"Torso", "Left Arm"}, {"Torso", "Right Arm"},
    {"Torso", "Left Leg"}, {"Torso", "Right Leg"},
}
local R15_BONES = {
    {"Head", "UpperTorso"}, {"UpperTorso", "LowerTorso"},
    {"UpperTorso", "LeftUpperArm"}, {"LeftUpperArm", "LeftLowerArm"}, {"LeftLowerArm", "LeftHand"},
    {"UpperTorso", "RightUpperArm"}, {"RightUpperArm", "RightLowerArm"}, {"RightLowerArm", "RightHand"},
    {"LowerTorso", "LeftUpperLeg"}, {"LeftUpperLeg", "LeftLowerLeg"}, {"LeftLowerLeg", "LeftFoot"},
    {"LowerTorso", "RightUpperLeg"}, {"RightUpperLeg", "RightLowerLeg"}, {"RightLowerLeg", "RightFoot"},
}

-- Detect rig type from a character
local function DetectRigType(char)
    if not char then return "Custom", R15_BONES end
    if char:FindFirstChild("UpperTorso") or char:FindFirstChild("LowerTorso") then
        return "R15", R15_BONES
    end
    if char:FindFirstChild("Torso") and char:FindFirstChild("Left Arm") then
        return "R6", R6_BONES
    end
    local hum = char:FindFirstChildOfClass("Humanoid")
    if hum and hum.RigType == Enum.HumanoidRigType.R6 then
        return "R6", R6_BONES
    end
    return "Custom", R15_BONES
end

-- Universal team check
local function SameTeam(target)
    if target == LP then return true end
    if not target then return false end
    if LP.Team and target.Team then
        if LP.Team == target.Team then return true end
        return false
    end
    local mc, tc = LP.TeamColor, target.TeamColor
    if mc and tc and mc == tc then
        local n = mc.Name
        if n ~= "Medium stone grey" and n ~= "Institutional white" then
            local count, total = 0, 0
            for _, p in ipairs(Players:GetPlayers()) do
                total = total + 1
                if p.TeamColor == mc then count = count + 1 end
            end
            if total > 0 and count / total <= 0.5 then
                return true
            end
        end
    end
    return false
end

-- Cache class
local CharCache = {}
CharCache.__index = CharCache

function CharCache.new(char)
    local self = setmetatable({}, CharCache)
    self.char = char
    self.parts = {}
    self.bones = R15_BONES
    self.rigType = "Custom"
    self.refresh()
    self.conns = {}
    table.insert(self.conns, char.DescendantAdded:Connect(function(d)
        if d:IsA("BasePart") then
            self.parts[d] = true
        end
    end))
    table.insert(self.conns, char.DescendantRemoving:Connect(function(d)
        if d:IsA("BasePart") then
            self.parts[d] = nil
        end
    end))
    return self
end

function CharCache:refresh()
    self.parts = {}
    for _, d in ipairs(self.char:GetDescendants()) do
        if d:IsA("BasePart") then
            self.parts[d] = true
        end
    end
    self.rigType, self.bones = DetectRigType(self.char)
end

function CharCache:getParts()
    local list = {}
    for p, _ in pairs(self.parts) do
        if p.Parent then
            table.insert(list, p)
        end
    end
    return list
end

function CharCache:getBoundingBox()
    local parts = self:getParts()
    if #parts == 0 then return nil end
    local minX, minY = math.huge, math.huge
    local maxX, maxY = -math.huge, -math.huge
    local any = false
    for _, part in ipairs(parts) do
        local sp = Camera:WorldToViewportPoint(part.Position)
        if sp.Z > 0 then
            any = true
            if sp.X < minX then minX = sp.X end
            if sp.Y < minY then minY = sp.Y end
            if sp.X > maxX then maxX = sp.X end
            if sp.Y > maxY then maxY = sp.Y end
        end
    end
    if not any then return nil end
    return minX, minY, maxX, maxY
end

function CharCache:destroy()
    for _, c in ipairs(self.conns) do
        pcall(function() c:Disconnect() end)
    end
    self.conns = {}
end

-- ============================================================
-- PLAYER INFO MODULE (internal)
-- ============================================================
local PInfo = {}
local Pin = setmetatable({}, { __index = function(t, k) return nil end })

local function PInfoRefresh(player)
    local info = PInfo[player]
    if not info then return end
    local char = player.Character
    info.alive = false
    info.hp = 0
    info.maxHp = 0
    info.dist = math.huge
    info.tool = "—"
    if not char then return end
    local hum = char:FindFirstChildOfClass("Humanoid")
    local root = char:FindFirstChild("HumanoidRootPart")
    if hum then
        info.hp = hum.Health
        info.maxHp = hum.MaxHealth
        info.alive = hum.Health > 0
    end
    if root then
        info.dist = (root.Position - Camera.CFrame.Position).Magnitude
    end
    local tool = char:FindFirstChildOfClass("Tool")
    if tool then
        info.tool = tool.Name
    else
        for _, c in ipairs(char:GetChildren()) do
            if c:IsA("Tool") then info.tool = c.Name break end
        end
    end
    info.team = player.Team
end

local function PInfoSetup(player)
    PInfo[player] = {
        hp = 0, maxHp = 0, alive = false, dist = 0,
        tool = "—", team = nil, lastHit = 0,
    }
end

-- ============================================================
-- ESP MODULE
-- ============================================================
local gParent = getHui()
local suffix  = tostring(math.random(100000, 999999))

local hudGui = Track(New("ScreenGui", gParent, {
    Name = "VodiumHUD_" .. suffix,
    DisplayOrder = 999998,
    IgnoreGuiInset = true,
    ResetOnSpawn = false,
}))
protectGui(hudGui)

local Esp = {
    cache = {},
    conns = {},
}

local function MakeLine(parent)
    return New("Frame", parent, {
        AnchorPoint = Vector2.new(0.5, 0.5),
        BackgroundColor3 = PAL.enemy,
        BorderSizePixel = 0,
        Visible = false,
        ZIndex = 3,
    })
end

local function SetLine(f, ax, ay, bx, by, thick)
    local dx, dy = bx - ax, by - ay
    local len = math.sqrt(dx * dx + dy * dy)
    local ang = math.deg(math.atan(dy, dx))
    f.Size = UDim2.new(0, len, 0, thick or 1.5)
    f.Position = UDim2.new(0, (ax + bx) * 0.5, 0, (ay + by) * 0.5)
    f.Rotation = ang
end

local function BuildEntry(player)
    local container = New("Frame", hudGui, {
        Size = UDim2.new(1, 0, 1, 0),
        BackgroundTransparency = 1,
        Visible = false,
        ZIndex = 2,
    })

    local box = {}
    for i = 1, 4 do
        box[i] = New("Frame", container, {
            BackgroundColor3 = PAL.enemy, BorderSizePixel = 0, ZIndex = 3, Visible = false,
        })
    end

    local corners = {}
    for i = 1, 8 do
        corners[i] = New("Frame", container, {
            BackgroundColor3 = PAL.enemy, BorderSizePixel = 0, ZIndex = 4, Visible = false,
        })
    end

    local fill = New("Frame", container, {
        BackgroundColor3 = PAL.enemy, BackgroundTransparency = 0.7,
        BorderSizePixel = 0, ZIndex = 2, Visible = false,
    })

    local hbBg = New("Frame", container, {
        BackgroundColor3 = Color3.fromRGB(30, 30, 30),
        BorderSizePixel = 0, ZIndex = 4, Visible = false,
    })
    local hbFill = New("Frame", hbBg, {
        BackgroundColor3 = PAL.ok, BorderSizePixel = 0, ZIndex = 5,
    })

    local hpText = New("TextLabel", container, {
        BackgroundTransparency = 1, TextColor3 = PAL.text, TextSize = 10,
        Font = Enum.Font.GothamBold, Text = "", ZIndex = 5, Visible = false,
    })

    local nameLabel = New("TextLabel", container, {
        BackgroundTransparency = 1, TextColor3 = PAL.text, TextSize = 11,
        Font = Enum.Font.GothamBold, Text = "", ZIndex = 5, Visible = false,
    })
    local dispLabel = New("TextLabel", container, {
        BackgroundTransparency = 1, TextColor3 = PAL.muted, TextSize = 10,
        Font = Enum.Font.Gotham, Text = "", ZIndex = 5, Visible = false,
    })
    local distLabel = New("TextLabel", container, {
        BackgroundTransparency = 1, TextColor3 = PAL.muted, TextSize = 10,
        Font = Enum.Font.Gotham, Text = "", ZIndex = 5, Visible = false,
    })
    local weapLabel = New("TextLabel", container, {
        BackgroundTransparency = 1, TextColor3 = PAL.warn, TextSize = 10,
        Font = Enum.Font.Gotham, Text = "", ZIndex = 5, Visible = false,
    })

    local dot = New("Frame", container, {
        AnchorPoint = Vector2.new(0.5, 0.5),
        BackgroundColor3 = PAL.enemy,
        BorderSizePixel = 0, ZIndex = 5, Visible = false,
        Size = UDim2.new(0, 6, 0, 6),
    })
    AddCorner(dot, 3)

    local tracer = MakeLine(container)

    local skel = {}
    for i = 1, 14 do skel[i] = MakeLine(container) end

    local arrow = New("Frame", container, {
        Size = UDim2.new(0, 14, 0, 14),
        AnchorPoint = Vector2.new(0.5, 0.5),
        BackgroundColor3 = PAL.enemy,
        BorderSizePixel = 0, ZIndex = 6, Visible = false,
        Rotation = 45,
    })
    AddCorner(arrow, 3)

    local cham = nil

    local entry = {
        player = player, container = container,
        box = box, corners = corners, fill = fill,
        hbBg = hbBg, hbFill = hbFill, hpText = hpText,
        name = nameLabel, disp = dispLabel, dist = distLabel, weap = weapLabel,
        dot = dot, tracer = tracer, skel = skel, arrow = arrow,
        cache = nil, cham = nil,
        lastDistInt = -1, lastHp = -1,
    }
    Esp.cache[player] = entry
    return entry
end

local function RebuildChar(entry)
    if entry.cache then
        entry.cache:destroy()
        entry.cache = nil
    end
    if entry.cham then
        pcall(function() entry.cham:Destroy() end)
        entry.cham = nil
    end
    local char = entry.player.Character
    if char then
        entry.cache = CharCache.new(char)
    end
end

local function RemoveEntry(player)
    local e = Esp.cache[player]
    if not e then return end
    if e.cache then e.cache:destroy() end
    if e.cham then pcall(function() e.cham:Destroy() end) end
    pcall(function() e.container:Destroy() end)
    Esp.cache[player] = nil
    if Esp.conns[player] then
        for _, c in ipairs(Esp.conns[player]) do
            pcall(function() c:Disconnect() end)
        end
        Esp.conns[player] = nil
    end
end

local function ApplyBox(e, x1, y1, x2, y2, th)
    local bw, bh = x2 - x1, y2 - y1
    e.box[1].Position = UDim2.new(0, x1, 0, y1)
    e.box[1].Size     = UDim2.new(0, bw, 0, th)
    e.box[2].Position = UDim2.new(0, x1, 0, y2 - th)
    e.box[2].Size     = UDim2.new(0, bw, 0, th)
    e.box[3].Position = UDim2.new(0, x1, 0, y1)
    e.box[3].Size     = UDim2.new(0, th, 0, bh)
    e.box[4].Position = UDim2.new(0, x2 - th, 0, y1)
    e.box[4].Size     = UDim2.new(0, th, 0, bh)
end

local function ApplyCorners(e, x1, y1, x2, y2, th, cLen)
    local bw, bh = x2 - x1, y2 - y1
    local cw = math.max(2, math.floor(bw * cLen))
    local ch = math.max(2, math.floor(bh * cLen))
    local c = e.corners
    c[1].Position = UDim2.new(0, x1, 0, y1);       c[1].Size = UDim2.new(0, cw, 0, th)
    c[2].Position = UDim2.new(0, x1, 0, y1);       c[2].Size = UDim2.new(0, th, 0, ch)
    c[3].Position = UDim2.new(0, x2 - cw, 0, y1);  c[3].Size = UDim2.new(0, cw, 0, th)
    c[4].Position = UDim2.new(0, x2 - th, 0, y1);  c[4].Size = UDim2.new(0, th, 0, ch)
    c[5].Position = UDim2.new(0, x1, 0, y2 - th);  c[5].Size = UDim2.new(0, cw, 0, th)
    c[6].Position = UDim2.new(0, x1, 0, y2 - ch);  c[6].Size = UDim2.new(0, th, 0, ch)
    c[7].Position = UDim2.new(0, x2 - cw, 0, y2 - th); c[7].Size = UDim2.new(0, cw, 0, th)
    c[8].Position = UDim2.new(0, x2 - th, 0, y2 - ch); c[8].Size = UDim2.new(0, th, 0, ch)
end

local function HideEntry(e)
    if e.container.Visible then e.container.Visible = false end
    if e.cham then pcall(function() e.cham:Destroy() end) e.cham = nil end
end

local espAccum = 0
local ESP_HZ = 1 / 60

local function RenderEsp(dt)
    if not Cfg.EspEnable then
        for _, e in pairs(Esp.cache) do HideEntry(e) end
        return
    end

    local cam = Camera
    if not cam then return end

    espAccum = espAccum + dt
    local hz = (#Players:GetPlayers() > 20) and (1 / 30) or ESP_HZ
    if espAccum < hz then return end
    espAccum = 0

    local camCF  = cam.CFrame
    local camPos = camCF.Position
    local camLook= camCF.LookVector
    local vp     = cam.ViewportSize
    local vpX, vpY = vp.X, vp.Y
    local cx, cy = vpX * 0.5, vpY * 0.5

    local cfg = Cfg

    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LP then
            local e = Esp.cache[player]
            if e then
                local char = player.Character
                local info = PInfo[player]
                if char and info and info.alive and e.cache then
                    local root = char:FindFirstChild("HumanoidRootPart") or char:FindFirstChild("Torso") or char:FindFirstChild("UpperTorso")
                    local hum  = char:FindFirstChildOfClass("Humanoid")
                    if root and hum then
                        local rootPos = root.Position
                        local toCam   = rootPos - camPos
                        local dist    = toCam.Magnitude

                        if toCam:Dot(camLook) >= -20
                            and dist <= cfg.EspMaxDist
                            and not (cfg.EspTeamCheck and SameTeam(player))
                        then
                            local x1, y1, x2, y2 = e.cache:getBoundingBox()
                            if x1 then
                                x1 = math.clamp(x1, -800, vpX + 800)
                                x2 = math.clamp(x2, -800, vpX + 800)
                                y1 = math.clamp(y1, -800, vpY + 800)
                                y2 = math.clamp(y2, -800, vpY + 800)
                                local bw, bh = x2 - x1, y2 - y1

                                if bw >= 1 and bh >= 1 and bw <= vpX * 3 then
                                    e.container.Visible = true

                                    local hp = math.clamp(hum.Health / math.max(hum.MaxHealth, 1), 0, 1)
                                    local baseColor = PAL.enemy
                                    if cfg.EspTeamColor and player.TeamColor then
                                        baseColor = player.TeamColor.Color
                                    end
                                    local hpColor = Color3.fromRGB((1 - hp) * 255, hp * 210, hp * 80)

                                    -- Box
                                    if cfg.EspBox and not cfg.EspCorner then
                                        ApplyBox(e, x1, y1, x2, y2, 1.5)
                                        for i = 1, 4 do
                                            e.box[i].BackgroundColor3 = baseColor
                                            e.box[i].Visible = true
                                        end
                                        for i = 1, 8 do e.corners[i].Visible = false end
                                    elseif cfg.EspCorner then
                                        ApplyCorners(e, x1, y1, x2, y2, 1.5, cfg.EspCornerLen)
                                        for i = 1, 8 do
                                            e.corners[i].BackgroundColor3 = baseColor
                                            e.corners[i].Visible = true
                                        end
                                        for i = 1, 4 do e.box[i].Visible = false end
                                    else
                                        for i = 1, 4 do e.box[i].Visible = false end
                                        for i = 1, 8 do e.corners[i].Visible = false end
                                    end

                                    -- Fill
                                    if cfg.EspFill then
                                        e.fill.Position = UDim2.new(0, x1, 0, y1)
                                        e.fill.Size     = UDim2.new(0, bw, 0, bh)
                                        e.fill.BackgroundColor3 = baseColor
                                        e.fill.Visible = true
                                    else
                                        e.fill.Visible = false
                                    end

                                    -- Health bar
                                    if cfg.EspHealthBar then
                                        local hbH = bh
                                        local filledH = math.floor(hbH * hp)
                                        e.hbBg.Position = UDim2.new(0, x1 - 6, 0, y1)
                                        e.hbBg.Size     = UDim2.new(0, 3, 0, hbH)
                                        e.hbBg.Visible  = true
                                        e.hbFill.Position = UDim2.new(0, 0, 1, -filledH)
                                        e.hbFill.Size     = UDim2.new(1, 0, 0, filledH)
                                        e.hbFill.BackgroundColor3 = hpColor
                                    else
                                        e.hbBg.Visible = false
                                    end

                                    -- Health text
                                    if cfg.EspHealthText then
                                        local hpInt = math.floor(hp * 100)
                                        if hpInt ~= e.lastHp then
                                            e.hpText.Text = hpInt .. ""
                                            e.lastHp = hpInt
                                        end
                                        e.hpText.Position = UDim2.new(0, x2 + 4, 0, y1)
                                        e.hpText.Size     = UDim2.new(0, 30, 0, 12)
                                        e.hpText.Visible  = true
                                    else
                                        e.hpText.Visible = false
                                    end

                                    -- Name
                                    if cfg.EspNames then
                                        local nameY = y1 - (cfg.EspDisplayName and 26 or 14)
                                        e.name.Position = UDim2.new(0, x1, 0, nameY)
                                        e.name.Size     = UDim2.new(0, bw, 0, 12)
                                        e.name.Text     = player.Name
                                        e.name.Visible  = true
                                    else
                                        e.name.Visible = false
                                    end

                                    -- Display name
                                    if cfg.EspDisplayName then
                                        e.disp.Position = UDim2.new(0, x1, 0, y1 - 14)
                                        e.disp.Size     = UDim2.new(0, bw, 0, 12)
                                        e.disp.Text     = player.DisplayName
                                        e.disp.Visible  = true
                                    else
                                        e.disp.Visible = false
                                    end

                                    -- Distance
                                    if cfg.EspDistance then
                                        local di = math.floor(dist)
                                        if di ~= e.lastDistInt then
                                            e.dist.Text = di >= 1000 and string.format("%.1fkm", di / 1000) or (di .. "m")
                                            e.lastDistInt = di
                                        end
                                        e.dist.Position = UDim2.new(0, x1, 0, y2 + 2)
                                        e.dist.Size     = UDim2.new(0, bw, 0, 12)
                                        e.dist.Visible  = true
                                    else
                                        e.dist.Visible = false
                                    end

                                    -- Weapon
                                    if cfg.EspWeapon then
                                        e.weap.Position = UDim2.new(0, x1, 0, y2 + 14)
                                        e.weap.Size     = UDim2.new(0, bw, 0, 12)
                                        e.weap.Text     = info.tool or "—"
                                        e.weap.Visible  = true
                                    else
                                        e.weap.Visible = false
                                    end

                                    -- Head dot
                                    local head = char:FindFirstChild("Head")
                                    if cfg.EspHeadDot and head then
                                        local hs, on = cam:WorldToViewportPoint(head.Position)
                                        if hs.Z > 0 and on then
                                            e.dot.Position = UDim2.new(0, hs.X, 0, hs.Y)
                                            e.dot.BackgroundColor3 = baseColor
                                            e.dot.Visible = true
                                        else
                                            e.dot.Visible = false
                                        end
                                    else
                                        e.dot.Visible = false
                                    end

                                    -- Tracers
                                    if cfg.EspTracers then
                                        local oy
                                        if cfg.EspTracerOrigin == "Top" then oy = 0
                                        elseif cfg.EspTracerOrigin == "Center" then oy = cy
                                        else oy = vpY end
                                        local tx = (x1 + x2) * 0.5
                                        local ty = y2
                                        SetLine(e.tracer, cx, oy, tx, ty, 1.5)
                                        e.tracer.BackgroundColor3 = baseColor
                                        e.tracer.Visible = true
                                    else
                                        e.tracer.Visible = false
                                    end

                                    -- Skeleton
                                    if cfg.EspSkeleton then
                                        local bones = e.cache.bones
                                        for i, b in ipairs(bones) do
                                            local s = e.skel[i]
                                            if s then
                                                local pa = char:FindFirstChild(b[1])
                                                local pb = char:FindFirstChild(b[2])
                                                if pa and pb then
                                                    local spa = cam:WorldToViewportPoint(pa.Position)
                                                    local spb = cam:WorldToViewportPoint(pb.Position)
                                                    if spa.Z > 0 and spb.Z > 0 then
                                                        SetLine(s, spa.X, spa.Y, spb.X, spb.Y, cfg.EspSkelThick)
                                                        s.BackgroundColor3 = baseColor
                                                        s.Visible = true
                                                    else
                                                        s.Visible = false
                                                    end
                                                else
                                                    s.Visible = false
                                                end
                                            end
                                        end
                                        for i = #bones + 1, 14 do
                                            if e.skel[i] then e.skel[i].Visible = false end
                                        end
                                    else
                                        for i = 1, 14 do
                                            if e.skel[i] then e.skel[i].Visible = false end
                                        end
                                    end

                                    -- Chams
                                    if cfg.EspChams then
                                        if not e.cham or e.cham.Parent ~= char then
                                            if e.cham then pcall(function() e.cham:Destroy() end) end
                                            local h = Instance.new("Highlight")
                                            h.FillColor = baseColor
                                            h.OutlineColor = baseColor
                                            h.FillTransparency = 0.6
                                            h.OutlineTransparency = 0
                                            h.Parent = char
                                            e.cham = h
                                        else
                                            e.cham.FillColor = baseColor
                                        end
                                    elseif e.cham then
                                        pcall(function() e.cham:Destroy() end)
                                        e.cham = nil
                                    end

                                    -- Off-screen arrow
                                    if cfg.EspOffscreen then
                                        local onScreen = x1 >= 0 and x2 <= vpX and y1 >= 0 and y2 <= vpY
                                        if not onScreen then
                                            local screenPos = cam:WorldToViewportPoint(rootPos)
                                            local dir
                                            if screenPos.Z < 0 then
                                                dir = Vector2.new(cx - screenPos.X, cy - screenPos.Y)
                                            else
                                                dir = Vector2.new(screenPos.X - cx, screenPos.Y - cy)
                                            end
                                            if dir.Magnitude > 0 then
                                                dir = dir.Unit
                                                local R = math.min(vpX, vpY) * 0.35
                                                local ax = cx + dir.X * R
                                                local ay = cy + dir.Y * R
                                                local ang = math.deg(math.atan(dir.Y, dir.X))
                                                e.arrow.Position = UDim2.new(0, ax, 0, ay)
                                                e.arrow.Rotation = ang + 45
                                                e.arrow.BackgroundColor3 = baseColor
                                                e.arrow.Visible = true
                                            end
                                        else
                                            e.arrow.Visible = false
                                        end
                                    else
                                        e.arrow.Visible = false
                                    end

                                    -- Visibility dot
                                    if cfg.EspVisibleDot then
                                        e.dot.Visible = true
                                        e.dot.BackgroundColor3 = Color3.fromRGB(60, 220, 110)
                                    end
                                else
                                    HideEntry(e)
                                end
                            else
                                HideEntry(e)
                            end
                        else
                            HideEntry(e)
                        end
                    else
                        HideEntry(e)
                    end
                else
                    HideEntry(e)
                end
            end
        end
    end
end

-- ESP player setup
local function SetupEspPlayer(player)
    if player == LP then return end
    local e = BuildEntry(player)
    local conns = {}
    table.insert(conns, player.CharacterAdded:Connect(function()
        task.wait(0.05)
        RebuildChar(e)
    end))
    table.insert(conns, player.CharacterRemoving:Connect(function()
        if e.cache then e.cache:destroy() e.cache = nil end
        if e.cham then pcall(function() e.cham:Destroy() end) e.cham = nil end
    end))
    Esp.conns[player] = conns
    if player.Character then
        RebuildChar(e)
    end
end

-- ============================================================
-- AIMBOT MODULE
-- ============================================================
local aimTarget = nil

local function GetAimPos(player)
    local char = player.Character
    if not char then return nil, nil end
    local bone = Cfg.AimBone
    local part
    if bone == "Head" then
        part = char:FindFirstChild("Head")
    elseif bone == "Torso" then
        part = char:FindFirstChild("UpperTorso") or char:FindFirstChild("Torso")
    else
        local best, bd = nil, math.huge
        local ctr = ScreenCenter()
        for _, p in ipairs(char:GetDescendants()) do
            if p:IsA("BasePart") then
                local sp = Camera:WorldToViewportPoint(p.Position)
                if sp.Z > 0 then
                    local d = (Vector2.new(sp.X, sp.Y) - ctr).Magnitude
                    if d < bd then bd = d; best = p end
                end
            end
        end
        part = best
    end
    if part then return part.Position, part end
    return nil, nil
end

local function WallCheck(targetPos, targetChar)
    local origin = Camera.CFrame.Position
    local dir    = targetPos - origin
    local rp     = RaycastParams.new()
    local ok = pcall(function() rp.FilterType = Enum.RaycastFilterType.Exclude end)
    if not ok then
        pcall(function() rp.FilterType = Enum.RaycastFilterType.Blacklist end)
    end
    local chars = { targetChar }
    if LP.Character then table.insert(chars, LP.Character) end
    rp.FilterDescendantsInstances = chars
    local result = Workspace:Raycast(origin, dir, rp)
    return result == nil
end

local function SelectAimTarget(useRage)
    local cam = Camera
    if not cam then return nil end
    local camPos = cam.CFrame.Position
    local ctr = ScreenCenter()
    local fov = useRage and Cfg.RageFOV or Cfg.AimFOV
    local best, bd = nil, fov

    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LP
            and not (Cfg.AimTeamCheck and SameTeam(player))
        then
            local char = player.Character
            local info = PInfo[player]
            if char and info and info.alive then
                local pos, part = GetAimPos(player)
                if pos then
                    local wd = (pos - camPos).Magnitude
                    if wd <= Cfg.AimMaxDist then
                        local visible = WallCheck(pos, char)
                        if (Cfg.AimWallCheck and visible) or (useRage and Cfg.RageWallPen) or (not Cfg.AimWallCheck) then
                            local sp = cam:WorldToViewportPoint(pos)
                            if sp.Z > 0 then
                                local sd = (Vector2.new(sp.X, sp.Y) - ctr).Magnitude
                                if Cfg.RageAim and Cfg.RagePriority == "Health" and info.hp < 30 then
                                    sd = sd * 0.3
                                elseif Cfg.RageAim and Cfg.RagePriority == "Distance" then
                                    sd = sd * 0.7 + wd * 0.3
                                end
                                if sd < bd then
                                    bd = sd
                                    best = player
                                end
                            end
                        end
                    end
                end
            end
        end
    end
    return best
end

local function RunAimbot(dt)
    if not (Cfg.AimEnable or Cfg.RageAim) then
        aimTarget = nil
        return
    end
    if not Cfg.RageAim and not AimKeyHeld() then
        aimTarget = nil
        return
    end

    aimTarget = SelectAimTarget(Cfg.RageAim)
    if not aimTarget then return end

    local pos = GetAimPos(aimTarget)
    if not pos then return end

    if Cfg.AimPredict then
        local char = aimTarget.Character
        local root = char and (char:FindFirstChild("HumanoidRootPart") or char:FindFirstChild("Torso"))
        if root then
            local vel = root.AssemblyLinearVelocity
            local d = (pos - Camera.CFrame.Position).Magnitude
            local scale = Cfg.AimPredScale or 1.0
            local grav = Workspace.Gravity or 196.2
            local bspeed = Cfg.AimBulletSpeed or 600
            local t = d / bspeed
            local drop = 0.5 * grav * t * t
            pos = pos + vel * t * scale + Vector3.new(0, drop, 0)
        end
    end

    local cam = Camera
    local camCF = cam.CFrame
    local origin = camCF.Position
    local dir = (pos - origin).Unit
    local lv = camCF.LookVector

    local alpha
    if Cfg.RageAim then
        alpha = 1
    else
        alpha = math.clamp(1 - Cfg.AimSmooth, 0.02, 1)
    end
    local newDir = lv:Lerp(dir, alpha)
    cam.CFrame = CFrame.new(origin, origin + newDir)
end

-- ============================================================
-- RAGE / HVH MODULE
-- ============================================================
local autoFireAccum = 0
local rageAutoFireRate = 1 / 15

local function RunAutoFire(dt)
    if not Cfg.RageAutoFire then return end
    if not aimTarget then return end
    autoFireAccum = autoFireAccum + dt
    if autoFireAccum >= rageAutoFireRate then
        autoFireAccum = 0
        ClickMouse()
    end
end

local function RunAutoReload()
    if not Cfg.RageAutoReload then return end
    local char = LP.Character
    if not char then return end
    local tool = char:FindFirstChildOfClass("Tool")
    if not tool then return end
    local ammo = tool:FindFirstChild("Ammo") or tool:FindFirstChild("Mag") or tool:FindFirstChild("Clip")
    if ammo and ammo:IsA("IntValue") and ammo.Value <= 0 then
        pcall(function()
            local VU = game:GetService("VirtualUser")
            VU:Button2Down(Vector2.new(0, 0), Camera.CFrame)
            task.wait(0.05)
            VU:Button2Up(Vector2.new(0, 0), Camera.CFrame)
        end)
    end
end

local antiFlingLast = 0
local function CheckAntiFling(dt)
    if not Cfg.AntiFling then return end
    local char = LP.Character
    if not char then return end
    local root = char:FindFirstChild("HumanoidRootPart")
    if root and root.AssemblyLinearVelocity.Magnitude > 200 then
        root.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
        root.AssemblyAngularVelocity = Vector3.new(0, 0, 0)
    end
end

local antiVoidLast = 0
local function CheckAntiVoid()
    if not Cfg.AntiVoid then return end
    local char = LP.Character
    if not char then return end
    local root = char:FindFirstChild("HumanoidRootPart")
    if root and root.Position.Y < -500 then
        local spawn = Workspace:FindFirstChildOfClass("SpawnLocation")
        local dest = spawn and spawn.CFrame or CFrame.new(0, 20, 0)
        root.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
        root.AssemblyAngularVelocity = Vector3.new(0, 0, 0)
        root.CFrame = dest + Vector3.new(0, 5, 0)
    end
end

-- ============================================================
-- TELEPORT MODULE
-- ============================================================
local Teleport = {}

function Teleport.ToPlayer(target)
    if not target then return Notify("Teleport", "No target", 2) end
    local myChar = LP.Character
    local myRoot = myChar and myChar:FindFirstChild("HumanoidRootPart")
    if not myRoot then return Notify("Teleport", "No local root", 2) end
    local tChar = target.Character
    local tRoot = tChar and tChar:FindFirstChild("HumanoidRootPart")
    if not tRoot then return Notify("Teleport", "Target has no root", 2) end
    pcall(function()
        myRoot.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
        myRoot.AssemblyAngularVelocity = Vector3.new(0, 0, 0)
    end)
    task.wait(0.05)
    myRoot.CFrame = tRoot.CFrame + Vector3.new(0, 3, 0)
    pcall(function()
        myRoot.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
    end)
    Notify("Teleport", "Teleported to " .. target.Name, 2)
end

function Teleport.ToMouse()
    local myChar = LP.Character
    local myRoot = myChar and myChar:FindFirstChild("HumanoidRootPart")
    if not myRoot then return Notify("Teleport", "No local root", 2) end
    local mouse = LP:GetMouse()
    if not mouse then return end
    local unitRay = Camera:ScreenPointToRay(mouse.X, mouse.Y)
    local rp = RaycastParams.new()
    pcall(function() rp.FilterType = Enum.RaycastFilterType.Exclude end)
    rp.FilterDescendantsInstances = { myChar }
    local res = Workspace:Raycast(unitRay.Origin, unitRay.Direction * 1000, rp)
    if not res then return Notify("Teleport", "No surface hit", 2) end
    pcall(function()
        myRoot.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
    end)
    myRoot.CFrame = CFrame.new(res.Position + Vector3.new(0, 5, 0))
    Notify("Teleport", "Teleported to mouse", 2)
end

function Teleport.ToWaypoint()
    local myChar = LP.Character
    local myRoot = myChar and myChar:FindFirstChild("HumanoidRootPart")
    if not myRoot then return Notify("Teleport", "No local root", 2) end
    if Cfg.TpDest then
        local dest = Cfg.TpDest
        Cfg.TpDest = nil
        pcall(function()
            myRoot.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
        end)
        myRoot.CFrame = dest
        Notify("Teleport", "Returned to waypoint", 2)
    else
        Cfg.TpDest = myRoot.CFrame
        Notify("Teleport", "Waypoint saved", 2)
    end
end

-- ============================================================
-- HITSOUND MODULE
-- ============================================================
local Hitsound = {
    lastPlayed = 0,
    sounds = {},
}

local function PlayHitSound(isHeadshot)
    if not Cfg.HitEnable then return end
    local now = os.clock()
    if now - Hitsound.lastPlayed < 0.05 then return end
    Hitsound.lastPlayed = now

    local id
    if isHeadshot then
        id = Cfg.HitStyle == "Rust" and DEFAULT_HITSOUND or DEFAULT_SKEET
    else
        id = Cfg.HitStyle == "Rust" and DEFAULT_HITSOUND or DEFAULT_SKEET
    end

    local snd = Instance.new("Sound")
    snd.SoundId = id
    snd.Volume = Cfg.HitVolume
    if Cfg.HitRage then
        snd.PlaybackSpeed = 0.75
        snd.Volume = Cfg.HitVolume * 1.3
    end
    pcall(function() SoundService:PlayLocalSound(snd) end)
    task.delay(3, function() pcall(function() snd:Destroy() end) end)
end

local function SetupHitsound(player)
    if player == LP then return end
    local conn
    conn = player.CharacterAdded:Connect(function(char)
        task.wait(0.2)
        local hum = char:FindFirstChildOfClass("Humanoid")
        if not hum then return end
        local lastHealth = hum.Health
        local c = hum.HealthChanged:Connect(function(hp)
            if hp < lastHealth then
                local root = char:FindFirstChild("HumanoidRootPart") or char:FindFirstChild("Torso")
                if root then
                    local dist = (root.Position - Camera.CFrame.Position).Magnitude
                    if dist <= 15 then
                        PlayHitSound(false)
                    end
                end
            end
            lastHealth = hp
        end)
        table.insert(V.Connections, c)
    end)
    table.insert(V.Connections, conn)
    if player.Character then
        local hum = player.Character:FindFirstChildOfClass("Humanoid")
        if hum then
            local lastHealth = hum.Health
            local c = hum.HealthChanged:Connect(function(hp)
                if hp < lastHealth then
                    local root = player.Character and (player.Character:FindFirstChild("HumanoidRootPart") or player.Character:FindFirstChild("Torso"))
                    if root then
                        local dist = (root.Position - Camera.CFrame.Position).Magnitude
                        if dist <= 15 then PlayHitSound(false) end
                    end
                end
                lastHealth = hp
            end)
            table.insert(V.Connections, c)
        end
    end
end

-- ============================================================
-- CROSSHAIR MODULE
-- ============================================================
local crossGui = Track(New("ScreenGui", gParent, {
    Name = "VodiumCrosshair_" .. suffix,
    DisplayOrder = 999999,
    IgnoreGuiInset = true,
    ResetOnSpawn = false,
}))
protectGui(crossGui)

local chContainer = New("Frame", crossGui, {
    Size = UDim2.new(0, 100, 0, 100),
    AnchorPoint = Vector2.new(0.5, 0.5),
    Position = UDim2.new(0.5, 0, 0.5, 0),
    BackgroundTransparency = 1,
    Visible = false,
    ZIndex = 50,
})

local chParts = {}
for i = 1, 4 do
    chParts[i] = New("Frame", chContainer, {
        BackgroundColor3 = PAL.white,
        BorderSizePixel = 0,
        ZIndex = 51,
    })
end
local chDot = New("Frame", chContainer, {
    BackgroundColor3 = PAL.white,
    BorderSizePixel = 0,
    ZIndex = 51,
    Size = UDim2.new(0, 2, 0, 2),
    AnchorPoint = Vector2.new(0.5, 0.5),
    Position = UDim2.new(0.5, 0, 0.5, 0),
})
AddCorner(chDot, 1)

local function ApplyCrosshair()
    chContainer.Visible = Cfg.ChEnable
    if not Cfg.ChEnable then return end
    local c = Color3.fromRGB(Cfg.ChColor[1], Cfg.ChColor[2], Cfg.ChColor[3])
    local size = Cfg.ChSize
    local thick = Cfg.ChThick
    local gap = Cfg.ChGap
    chParts[1].BackgroundColor3 = c
    chParts[2].BackgroundColor3 = c
    chParts[3].BackgroundColor3 = c
    chParts[4].BackgroundColor3 = c
    chParts[1].Size = UDim2.new(0, size, 0, thick)
    chParts[1].Position = UDim2.new(0.5, 0, 0.5, -gap - thick)
    chParts[1].AnchorPoint = Vector2.new(0.5, 0)
    chParts[2].Size = UDim2.new(0, size, 0, thick)
    chParts[2].Position = UDim2.new(0.5, 0, 0.5, gap)
    chParts[2].AnchorPoint = Vector2.new(0.5, 0)
    chParts[3].Size = UDim2.new(0, thick, 0, size)
    chParts[3].Position = UDim2.new(0.5, -gap - thick, 0.5, 0)
    chParts[3].AnchorPoint = Vector2.new(0, 0.5)
    chParts[4].Size = UDim2.new(0, thick, 0, size)
    chParts[4].Position = UDim2.new(0.5, gap, 0.5, 0)
    chParts[4].AnchorPoint = Vector2.new(0, 0.5)
    chDot.BackgroundColor3 = c
    chDot.Visible = Cfg.ChDot
end

-- ============================================================
-- DAMAGE INDICATOR
-- ============================================================
local dmgGui = Track(New("ScreenGui", gParent, {
    Name = "VodiumDmg_" .. suffix,
    DisplayOrder = 999997,
    IgnoreGuiInset = true,
    ResetOnSpawn = false,
}))
protectGui(dmgGui)

local dmgArcs = {}
for i = 1, 8 do
    local arc = New("Frame", dmgGui, {
        Size = UDim2.new(0, 40, 0, 8),
        AnchorPoint = Vector2.new(0.5, 0.5),
        BackgroundColor3 = PAL.err,
        BackgroundTransparency = 1,
        BorderSizePixel = 0,
        Visible = false,
        ZIndex = 40,
    })
    AddCorner(arc, 2)
    dmgArcs[i] = { frame = arc, alpha = 0 }
end

local function ShowDamageIndicator(fromPos)
    local vp = Camera.ViewportSize
    local ctr = Vector2.new(vp.X * 0.5, vp.Y * 0.5)
    local sp = Camera:WorldToViewportPoint(fromPos)
    local dir = Vector2.new(sp.X - ctr.X, sp.Y - ctr.Y)
    if dir.Magnitude < 1 then return end
    dir = dir.Unit
    local R = math.min(vp.X, vp.Y) * 0.22
    local idx = 1
    for i = 1, 8 do
        if dmgArcs[i].alpha <= 0 then idx = i break end
    end
    local arc = dmgArcs[idx]
    local ax = ctr.X + dir.X * R
    local ay = ctr.Y + dir.Y * R
    local ang = math.deg(math.atan(dir.Y, dir.X))
    arc.frame.Position = UDim2.new(0, ax, 0, ay)
    arc.frame.Rotation = ang
    arc.frame.Visible = true
    arc.frame.BackgroundTransparency = 0.1
    arc.alpha = 1
end

local function UpdateDamageIndicators(dt)
    for i = 1, 8 do
        local a = dmgArcs[i]
        if a.alpha > 0 then
            a.alpha = a.alpha - dt * 1.25
            if a.alpha <= 0 then
                a.alpha = 0
                a.frame.Visible = false
            else
                a.frame.BackgroundTransparency = 1 - a.alpha * 0.9
            end
        end
    end
end

-- ============================================================
-- RADAR MODULE
-- ============================================================
local radarGui = Track(New("ScreenGui", gParent, {
    Name = "VodiumRadar_" .. suffix,
    DisplayOrder = 999996,
    IgnoreGuiInset = true,
    ResetOnSpawn = false,
}))
protectGui(radarGui)

local radarRoot = New("Frame", radarGui, {
    Size = UDim2.new(0, 180, 0, 180),
    AnchorPoint = Vector2.new(1, 1),
    Position = UDim2.new(1, -14, 1, -14),
    BackgroundColor3 = PAL.bg,
    BackgroundTransparency = 0.35,
    Visible = false,
    ZIndex = 30,
    ClipsDescendants = true,
})
AddCorner(radarRoot, 90)
AddStroke(radarRoot, PAL.accent1, 2, 0.3)

local radarPlayers = {}

local function UpdateRadar()
    radarRoot.Visible = Cfg.RadarEnable
    if not Cfg.RadarEnable then
        for _, f in pairs(radarPlayers) do f.Visible = false end
        return
    end
    radarRoot.Size = UDim2.new(0, Cfg.RadarRadius * 2, 0, Cfg.RadarRadius * 2)
    radarRoot.BackgroundTransparency = 1 - Cfg.RadarOpacity

    local cam = Camera
    if not cam then return end
    local camCF = cam.CFrame
    local camPos = camCF.Position
    local look = camCF.LookVector
    local right = camCF.RightVector

    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LP then
            local info = PInfo[player]
            local char = player.Character
            local root = char and (char:FindFirstChild("HumanoidRootPart") or char:FindFirstChild("Torso"))
            local f = radarPlayers[player]
            if not f then
                f = New("Frame", radarRoot, {
                    Size = UDim2.new(0, 5, 0, 5),
                    AnchorPoint = Vector2.new(0.5, 0.5),
                    BackgroundColor3 = PAL.enemy,
                    BorderSizePixel = 0,
                    ZIndex = 32,
                    Visible = false,
                })
                AddCorner(f, 3)
                radarPlayers[player] = f
            end
            if info and info.alive and root and not (Cfg.EspTeamCheck and SameTeam(player)) then
                local rel = root.Position - camPos
                local x = rel:Dot(right)
                local z = rel:Dot(look)
                local scale = Cfg.RadarScale
                local px = Cfg.RadarRadius + x / scale
                local py = Cfg.RadarRadius - z / scale
                local cx = Cfg.RadarRadius
                local cy = Cfg.RadarRadius
                local dx, dy = px - cx, py - cy
                local d = math.sqrt(dx * dx + dy * dy)
                if d > Cfg.RadarRadius - 4 then
                    local scale2 = (Cfg.RadarRadius - 4) / d
                    dx, dy = dx * scale2, dy * scale2
                end
                f.Position = UDim2.new(0, cx + dx, 0, cy + dy)
                f.BackgroundColor3 = SameTeam(player) and PAL.ally or PAL.enemy
                f.Visible = true
            else
                f.Visible = false
            end
        end
    end
end

-- ============================================================
-- PLAYER LIST PANEL
-- ============================================================
local plistGui = Track(New("ScreenGui", gParent, {
    Name = "VodiumPlist_" .. suffix,
    DisplayOrder = 999995,
    IgnoreGuiInset = true,
    ResetOnSpawn = false,
}))
protectGui(plistGui)

local plistRoot = New("Frame", plistGui, {
    Size = UDim2.new(0, 220, 0, 300),
    Position = UDim2.new(0, 10, 0, 60),
    BackgroundColor3 = PAL.panel,
    BackgroundTransparency = 0.1,
    Visible = false,
    ZIndex = 30,
})
AddCorner(plistRoot, 8)
AddStroke(plistRoot, PAL.stroke, 1, 0.2)

New("TextLabel", plistRoot, {
    Size = UDim2.new(1, 0, 0, 22),
    BackgroundTransparency = 1,
    Text = "PLAYERS",
    TextColor3 = PAL.muted,
    TextSize = 11,
    Font = Enum.Font.GothamBold,
    TextXAlignment = Enum.TextXAlignment.Left,
    ZIndex = 31,
    Position = UDim2.new(0, 10, 0, 4),
})

local plistScroll = New("ScrollingFrame", plistRoot, {
    Size = UDim2.new(1, -8, 1, -32),
    Position = UDim2.new(0, 4, 0, 28),
    BackgroundTransparency = 1,
    BorderSizePixel = 0,
    ScrollBarThickness = 3,
    ScrollBarImageColor3 = PAL.accent1,
    CanvasSize = UDim2.new(0, 0, 0, 0),
    ZIndex = 31,
})
New("UIListLayout", plistScroll, {
    Padding = UDim.new(0, 3),
    SortOrder = Enum.SortOrder.LayoutOrder,
})

local plistRows = {}

local function UpdatePlayerList()
    plistRoot.Visible = Cfg.PlistEnable
    if not Cfg.PlistEnable then return end

    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= LP then
            local row = plistRows[p]
            if not row then
                row = New("TextButton", plistScroll, {
                    Size = UDim2.new(1, 0, 0, 24),
                    BackgroundColor3 = PAL.card,
                    BackgroundTransparency = 0.5,
                    Text = "",
                    ZIndex = 32,
                })
                AddCorner(row, 4)
                local label = New("TextLabel", row, {
                    Size = UDim2.new(1, -8, 1, 0),
                    Position = UDim2.new(0, 4, 0, 0),
                    BackgroundTransparency = 1,
                    Text = "",
                    TextColor3 = PAL.text,
                    TextSize = 10,
                    Font = Enum.Font.Gotham,
                    TextXAlignment = Enum.TextXAlignment.Left,
                    ZIndex = 33,
                })
                row.MouseButton1Click:Connect(function()
                    Teleport.ToPlayer(p)
                end)
                plistRows[p] = { row = row, label = label }
            end
            local info = PInfo[p]
            if info then
                local teamStr = "FFA"
                if p.Team then teamStr = p.Team.Name end
                local hpStr = info.alive and string.format("%d%%", math.floor(info.hp / math.max(info.maxHp, 1) * 100)) or "DEAD"
                plistRows[p].label.Text = string.format("%s | %s | %dm | %s", p.Name, teamStr, math.floor(info.dist), hpStr)
                plistRows[p].label.TextColor3 = SameTeam(p) and PAL.ally or PAL.text
            end
        end
    end

    for p, data in pairs(plistRows) do
        if not p.Parent then
            data.row:Destroy()
            plistRows[p] = nil
        end
    end
end

-- ============================================================
-- VISUAL MODULE
-- ============================================================
local origLighting = {}
local lightingApplied = false

local function ApplyFullbright(on)
    if on and not lightingApplied then
        origLighting.Ambient       = Lighting.Ambient
        origLighting.OutdoorAmbient= Lighting.OutdoorAmbient
        origLighting.Brightness    = Lighting.Brightness
        origLighting.GlobalShadows = Lighting.GlobalShadows
        Lighting.Ambient           = Color3.fromRGB(180, 180, 180)
        Lighting.OutdoorAmbient    = Color3.fromRGB(180, 180, 180)
        Lighting.Brightness        = 2
        Lighting.GlobalShadows     = false
        lightingApplied = true
    elseif not on and lightingApplied then
        Lighting.Ambient           = origLighting.Ambient
        Lighting.OutdoorAmbient    = origLighting.OutdoorAmbient
        Lighting.Brightness        = origLighting.Brightness
        Lighting.GlobalShadows     = origLighting.GlobalShadows
        lightingApplied = false
    end
end

local origFog = {}
local fogApplied = false

local function ApplyNoFog(on)
    if on and not fogApplied then
        origFog.FogEnd   = Lighting.FogEnd
        origFog.FogStart = Lighting.FogStart
        origFog.FogColor = Lighting.FogColor
        Lighting.FogEnd   = 1e6
        Lighting.FogStart = 1e6 - 1
        fogApplied = true
    elseif not on and fogApplied then
        Lighting.FogEnd   = origFog.FogEnd
        Lighting.FogStart = origFog.FogStart
        Lighting.FogColor = origFog.FogColor
        fogApplied = false
    end
end

local function ApplyLowGfx(on)
    safeCall(function()
        if on then
            settings().Rendering.QualityLevel = Enum.QualityLevel.Level01
        else
            settings().Rendering.QualityLevel = Enum.QualityLevel.Automatic
        end
    end)
end

-- Spin visual
local spinGui = Track(New("ScreenGui", gParent, {
    Name = "VodiumSpin_" .. suffix,
    DisplayOrder = 999990,
    IgnoreGuiInset = true,
    ResetOnSpawn = false,
}))
protectGui(spinGui)

local spinFrame = New("Frame", spinGui, {
    Size = UDim2.new(0, 20, 0, 20),
    Position = UDim2.new(0, 14, 0.5, -10),
    BackgroundColor3 = PAL.accent1,
    BorderSizePixel = 0,
    Rotation = 45,
    Visible = false,
    ZIndex = 30,
})
AddCorner(spinFrame, 3)

local spinAngle = 45
local function UpdateSpin(dt)
    spinFrame.Visible = Cfg.SpinVisual
    if not Cfg.SpinVisual then return end
    spinAngle = spinAngle + Cfg.SpinSpeed * dt
    if spinAngle >= 405 then spinAngle = spinAngle - 360 end
    spinFrame.Rotation = spinAngle
end

-- ============================================================
-- MOVEMENT MODULE
-- ============================================================
local flyActive, flyBodyVel, flyBodyAng = false, nil, nil

local function SetFly(on)
    local char = LP.Character
    if not char then return end
    local root = char:FindFirstChild("HumanoidRootPart")
    local hum  = char:FindFirstChildOfClass("Humanoid")
    if not root or not hum then return end

    if on and not flyActive then
        flyActive = true
        hum.PlatformStand = true
        flyBodyVel = Instance.new("BodyVelocity")
        flyBodyVel.Velocity = Vector3.new(0, 0, 0)
        flyBodyVel.MaxForce = Vector3.new(1e5, 1e5, 1e5)
        flyBodyVel.P = 1e4
        flyBodyVel.Parent = root
        flyBodyAng = Instance.new("BodyAngularVelocity")
        flyBodyAng.AngularVelocity = Vector3.new(0, 0, 0)
        flyBodyAng.MaxTorque = Vector3.new(1e5, 1e5, 1e5)
        flyBodyAng.P = 1e4
        flyBodyAng.Parent = root
    elseif not on and flyActive then
        flyActive = false
        if flyBodyVel then flyBodyVel:Destroy() flyBodyVel = nil end
        if flyBodyAng then flyBodyAng:Destroy() flyBodyAng = nil end
        hum.PlatformStand = false
    end
end

local function UpdateFly()
    if not flyActive or not flyBodyVel then return end
    local cf = Camera.CFrame
    local s = Cfg.FlySpeed
    local v = Vector3.new(0, 0, 0)
    if UserInputService:IsKeyDown(Enum.KeyCode.W) then v = v + cf.LookVector * s end
    if UserInputService:IsKeyDown(Enum.KeyCode.S) then v = v - cf.LookVector * s end
    if UserInputService:IsKeyDown(Enum.KeyCode.A) then v = v - cf.RightVector * s end
    if UserInputService:IsKeyDown(Enum.KeyCode.D) then v = v + cf.RightVector * s end
    if UserInputService:IsKeyDown(Enum.KeyCode.Space) then v = v + Vector3.new(0, 1, 0) * s end
    if UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) then v = v - Vector3.new(0, 1, 0) * s end
    flyBodyVel.Velocity = v
end

local noclipActive = false
local function SetNoclip(on) noclipActive = on end
local function UpdateNoclip()
    if not noclipActive then return end
    local char = LP.Character
    if not char then return end
    for _, p in ipairs(char:GetDescendants()) do
        if p:IsA("BasePart") and p.CanCollide then
            p.CanCollide = false
        end
    end
end

local function UpdateSpeed()
    local char = LP.Character
    if not char then return end
    local hum = char:FindFirstChildOfClass("Humanoid")
    if not hum then return end
    hum.WalkSpeed = Cfg.SpeedHack and Cfg.WalkSpeed or 16
end

Conn(UserInputService.JumpRequest, function()
    if not Cfg.InfJump then return end
    local char = LP.Character
    if not char then return end
    local hum = char:FindFirstChildOfClass("Humanoid")
    if hum then hum:ChangeState(Enum.HumanoidStateType.Jumping) end
end)

-- ============================================================
-- MISC MODULE
-- ============================================================
local afkThread = nil
local function StartAntiAfk()
    if afkThread then task.cancel(afkThread) end
    afkThread = task.spawn(function()
        while not V.Unloaded do
            task.wait(180)
            if Cfg.AntiAfk then
                pcall(function()
                    local VU = game:GetService("VirtualUser")
                    VU:CaptureController()
                    VU:ClickButton2(Vector2.new(0, 0))
                end)
            end
        end
    end)
end

-- ============================================================
-- GUI (MAIN MENU)
-- ============================================================
local menuGui = Track(New("ScreenGui", gParent, {
    Name = "VodiumMenu_" .. suffix,
    DisplayOrder = 1000000,
    IgnoreGuiInset = true,
    ResetOnSpawn = false,
}))
protectGui(menuGui)

local menuScale = New("UIScale", menuGui, { Scale = Cfg.GuiScale })

local shadow = New("Frame", menuGui, {
    Size = UDim2.new(0, 804, 0, 564),
    Position = UDim2.new(0.5, -402, 0.5, -282),
    BackgroundColor3 = Color3.new(0, 0, 0),
    BackgroundTransparency = 0.5,
    ZIndex = 20,
})
AddCorner(shadow, 18)

local mainPanel = New("Frame", menuGui, {
    Size = UDim2.new(0, 780, 0, 540),
    Position = UDim2.new(0.5, -390, 0.5, -270),
    BackgroundColor3 = PAL.bg,
    ZIndex = 21,
    ClipsDescendants = true,
})
AddCorner(mainPanel, 14)
AddStroke(mainPanel, PAL.stroke, 1, 0.2)
New("UIGradient", mainPanel, {
    Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromRGB(12, 10, 22)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(8, 10, 20)),
    }),
    Rotation = 135,
})

-- Header
local header = New("Frame", mainPanel, {
    Size = UDim2.new(1, 0, 0, 52),
    BackgroundColor3 = PAL.panel,
    BackgroundTransparency = 0.3,
    ZIndex = 22,
})
AddStroke(header, PAL.stroke, 1, 0.5)

local diamond = New("Frame", header, {
    Size = UDim2.new(0, 26, 0, 26),
    Position = UDim2.new(0, 14, 0.5, -13),
    Rotation = 45,
    BackgroundColor3 = PAL.accent1,
    ZIndex = 23,
})
AddCorner(diamond, 6)
New("UIGradient", diamond, {
    Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, PAL.accent1),
        ColorSequenceKeypoint.new(1, PAL.accent2),
    }),
    Rotation = 135,
})

New("TextLabel", header, {
    Size = UDim2.new(0, 160, 0, 22),
    Position = UDim2.new(0, 48, 0, 8),
    BackgroundTransparency = 1,
    Text = "Vodium",
    TextColor3 = PAL.text,
    TextSize = 18,
    Font = Enum.Font.GothamBold,
    TextXAlignment = Enum.TextXAlignment.Left,
    ZIndex = 23,
})
New("TextLabel", header, {
    Size = UDim2.new(0, 80, 0, 16),
    Position = UDim2.new(0, 48, 0, 30),
    BackgroundTransparency = 1,
    Text = "v" .. SCRIPT_VERSION,
    TextColor3 = PAL.muted,
    TextSize = 11,
    Font = Enum.Font.Gotham,
    TextXAlignment = Enum.TextXAlignment.Left,
    ZIndex = 23,
})
New("TextLabel", header, {
    Size = UDim2.new(0, 220, 1, 0),
    Position = UDim2.new(1, -330, 0, 0),
    BackgroundTransparency = 1,
    Text = LP.DisplayName .. " · @" .. LP.Name,
    TextColor3 = PAL.muted,
    TextSize = 11,
    Font = Enum.Font.Gotham,
    TextXAlignment = Enum.TextXAlignment.Right,
    ZIndex = 23,
})

local minimizeBtn = New("TextButton", header, {
    Size = UDim2.new(0, 32, 0, 32),
    Position = UDim2.new(1, -74, 0.5, -16),
    BackgroundColor3 = PAL.card,
    Text = "—",
    TextColor3 = PAL.muted,
    TextSize = 14,
    Font = Enum.Font.GothamBold,
    ZIndex = 24,
})
AddCorner(minimizeBtn, 6)

local closeBtn = New("TextButton", header, {
    Size = UDim2.new(0, 32, 0, 32),
    Position = UDim2.new(1, -36, 0.5, -16),
    BackgroundColor3 = PAL.err,
    Text = "✕",
    TextColor3 = PAL.white,
    TextSize = 14,
    Font = Enum.Font.GothamBold,
    ZIndex = 24,
})
AddCorner(closeBtn, 6)

-- Sidebar
local sidebar = New("Frame", mainPanel, {
    Size = UDim2.new(0, 168, 1, -74),
    Position = UDim2.new(0, 0, 0, 52),
    BackgroundColor3 = PAL.panel,
    BackgroundTransparency = 0.5,
    ZIndex = 22,
})
AddStroke(sidebar, PAL.stroke, 1, 0.5)
New("UIListLayout", sidebar, {
    Padding = UDim.new(0, 3),
    FillDirection = Enum.FillDirection.Vertical,
    HorizontalAlignment = Enum.HorizontalAlignment.Center,
    SortOrder = Enum.SortOrder.LayoutOrder,
})
New("UIPadding", sidebar, {
    PaddingTop = UDim.new(0, 8),
    PaddingLeft = UDim.new(0, 6),
    PaddingRight = UDim.new(0, 6),
})

local contentArea = New("Frame", mainPanel, {
    Size = UDim2.new(1, -168, 1, -74),
    Position = UDim2.new(0, 168, 0, 52),
    BackgroundTransparency = 1,
    ZIndex = 22,
    ClipsDescendants = true,
})

local statusBar = New("Frame", mainPanel, {
    Size = UDim2.new(1, 0, 0, 22),
    Position = UDim2.new(0, 0, 1, -22),
    BackgroundColor3 = PAL.panel,
    BackgroundTransparency = 0.4,
    ZIndex = 23,
})
local statusLabel = New("TextLabel", statusBar, {
    Size = UDim2.new(1, -10, 1, 0),
    Position = UDim2.new(0, 10, 0, 0),
    BackgroundTransparency = 1,
    Text = "fps: -- · ping: -- · --:--",
    TextColor3 = PAL.muted,
    TextSize = 10,
    Font = Enum.Font.Gotham,
    TextXAlignment = Enum.TextXAlignment.Left,
    ZIndex = 24,
})

-- ============================================================
-- GUI (WIDGETS)
-- ============================================================
local tabAccents = {
    ESP = PAL.accent1, Aimbot = PAL.accent3, Rage = PAL.err,
    Teleport = PAL.accent2, Hitsounds = PAL.warn, Crosshair = PAL.ok,
    Visuals = PAL.accent1, Radar = PAL.accent2, Players = PAL.ok,
    Movement = PAL.accent2, Misc = PAL.muted, Config = PAL.accent1, Info = PAL.ok,
}
local tabPages, tabButtons, activeTab = {}, {}, nil

local function MakeTab(name, layoutOrder)
    local color = tabAccents[name] or PAL.accent1

    local btn = New("TextButton", sidebar, {
        Size = UDim2.new(1, 0, 0, 34),
        BackgroundColor3 = PAL.card,
        BackgroundTransparency = 1,
        Text = "",
        ZIndex = 23,
        LayoutOrder = layoutOrder or 0,
    })
    AddCorner(btn, 7)

    local indicator = New("Frame", btn, {
        Size = UDim2.new(0, 3, 0, 18),
        Position = UDim2.new(0, 0, 0.5, -9),
        BackgroundColor3 = color,
        BackgroundTransparency = 1,
        BorderSizePixel = 0,
        ZIndex = 24,
    })
    AddCorner(indicator, 2)

    New("TextLabel", btn, {
        Size = UDim2.new(1, -14, 1, 0),
        Position = UDim2.new(0, 14, 0, 0),
        BackgroundTransparency = 1,
        Text = name,
        TextColor3 = PAL.muted,
        TextSize = 12,
        Font = Enum.Font.GothamBold,
        TextXAlignment = Enum.TextXAlignment.Left,
        ZIndex = 24,
    })

    local page = New("Frame", contentArea, {
        Size = UDim2.new(1, 0, 1, 0),
        BackgroundTransparency = 1,
        Visible = false,
        ZIndex = 22,
        ClipsDescendants = true,
    })
    New("UIPadding", page, {
        PaddingTop = UDim.new(0, 10),
        PaddingLeft = UDim.new(0, 14),
        PaddingRight = UDim.new(0, 14),
        PaddingBottom = UDim.new(0, 10),
    })
    local layout = New("UIListLayout", page, {
        Padding = UDim.new(0, 6),
        FillDirection = Enum.FillDirection.Vertical,
        HorizontalAlignment = Enum.HorizontalAlignment.Left,
        SortOrder = Enum.SortOrder.LayoutOrder,
    })

    tabPages[name] = page
    tabButtons[name] = { btn = btn, indicator = indicator, color = color, layout = layout }

    btn.MouseEnter:Connect(function()
        if activeTab ~= name then
            btn.BackgroundColor3 = PAL.card
            Tween(btn, TweenInfo.new(0.1), { BackgroundTransparency = 0.5 })
        end
    end)
    btn.MouseLeave:Connect(function()
        if activeTab ~= name then
            Tween(btn, TweenInfo.new(0.1), { BackgroundTransparency = 1 })
        end
    end)

    btn.MouseButton1Click:Connect(function()
        if activeTab then
            tabPages[activeTab].Visible = false
            local pb = tabButtons[activeTab]
            Tween(pb.btn, TweenInfo.new(0.15), { BackgroundTransparency = 1 })
            Tween(pb.indicator, TweenInfo.new(0.15), { BackgroundTransparency = 1 })
            pb.btn.BackgroundColor3 = PAL.card
        end
        activeTab = name
        page.Visible = true
        btn.BackgroundColor3 = PAL.card
        Tween(btn, TweenInfo.new(0.15), { BackgroundTransparency = 0.5 })
        Tween(indicator, TweenInfo.new(0.15), { BackgroundTransparency = 0 })
    end)

    return page
end

local widgetCounter = 0
local function AddSection(page, label)
    widgetCounter = widgetCounter + 1
    local row = New("Frame", page, {
        Size = UDim2.new(1, 0, 0, 24),
        BackgroundTransparency = 1,
        ZIndex = 23,
        LayoutOrder = widgetCounter,
    })
    New("Frame", row, {
        Size = UDim2.new(0, 3, 0, 12),
        Position = UDim2.new(0, 0, 0.5, -6),
        BackgroundColor3 = PAL.accent1,
        BorderSizePixel = 0,
        ZIndex = 24,
    })
    New("TextLabel", row, {
        Size = UDim2.new(1, -10, 1, 0),
        Position = UDim2.new(0, 8, 0, 0),
        BackgroundTransparency = 1,
        Text = string.upper(label),
        TextColor3 = PAL.muted,
        TextSize = 10,
        Font = Enum.Font.GothamBold,
        TextXAlignment = Enum.TextXAlignment.Left,
        ZIndex = 24,
    })
    return row
end

local function AddToggle(page, label, cfgKey, onChange)
    widgetCounter = widgetCounter + 1
    local row = New("Frame", page, {
        Size = UDim2.new(1, 0, 0, 34),
        BackgroundColor3 = PAL.card,
        BackgroundTransparency = 0.5,
        ZIndex = 23,
        LayoutOrder = widgetCounter,
    })
    AddCorner(row, 7)

    New("TextLabel", row, {
        Size = UDim2.new(1, -56, 1, 0),
        Position = UDim2.new(0, 10, 0, 0),
        BackgroundTransparency = 1,
        Text = label,
        TextColor3 = PAL.text,
        TextSize = 12,
        Font = Enum.Font.Gotham,
        TextXAlignment = Enum.TextXAlignment.Left,
        ZIndex = 24,
    })

    local track = New("Frame", row, {
        Size = UDim2.new(0, 36, 0, 20),
        Position = UDim2.new(1, -46, 0.5, -10),
        BackgroundColor3 = PAL.elevated,
        ZIndex = 24,
    })
    AddCorner(track, 10)

    local knob = New("Frame", track, {
        Size = UDim2.new(0, 16, 0, 16),
        Position = UDim2.new(0, 2, 0.5, -8),
        BackgroundColor3 = PAL.text,
        ZIndex = 25,
    })
    AddCorner(knob, 8)

    local function setState(val, cb)
        Cfg[cfgKey] = val
        if val then
            Tween(track, TweenInfo.new(0.15), { BackgroundColor3 = PAL.ok })
            Tween(knob,  TweenInfo.new(0.15), { Position = UDim2.new(0, 18, 0.5, -8) })
        else
            Tween(track, TweenInfo.new(0.15), { BackgroundColor3 = PAL.elevated })
            Tween(knob,  TweenInfo.new(0.15), { Position = UDim2.new(0, 2, 0.5, -8) })
        end
        if cb and onChange then onChange(val) end
    end
    setState(Cfg[cfgKey] or false, false)

    local click = New("TextButton", row, {
        Size = UDim2.new(1, 0, 1, 0),
        BackgroundTransparency = 1,
        Text = "",
        ZIndex = 26,
    })
    click.MouseButton1Click:Connect(function()
        setState(not Cfg[cfgKey], true)
    end)

    return row, setState
end

local function AddSlider(page, label, cfgKey, minV, maxV, fmt, onChange)
    widgetCounter = widgetCounter + 1
    local row = New("Frame", page, {
        Size = UDim2.new(1, 0, 0, 48),
        BackgroundColor3 = PAL.card,
        BackgroundTransparency = 0.5,
        ZIndex = 23,
        LayoutOrder = widgetCounter,
    })
    AddCorner(row, 7)

    New("TextLabel", row, {
        Size = UDim2.new(0.6, 0, 0, 20),
        Position = UDim2.new(0, 10, 0, 6),
        BackgroundTransparency = 1,
        Text = label,
        TextColor3 = PAL.text,
        TextSize = 12,
        Font = Enum.Font.Gotham,
        TextXAlignment = Enum.TextXAlignment.Left,
        ZIndex = 24,
    })

    local badge = New("Frame", row, {
        Size = UDim2.new(0, 54, 0, 18),
        Position = UDim2.new(1, -64, 0, 6),
        BackgroundColor3 = PAL.elevated,
        ZIndex = 24,
    })
    AddCorner(badge, 5)
    local badgeLabel = New("TextLabel", badge, {
        Size = UDim2.new(1, 0, 1, 0),
        BackgroundTransparency = 1,
        Text = tostring(Cfg[cfgKey] or minV),
        TextColor3 = PAL.text,
        TextSize = 11,
        Font = Enum.Font.GothamBold,
        ZIndex = 25,
    })

    local trackBg = New("Frame", row, {
        Size = UDim2.new(1, -20, 0, 6),
        Position = UDim2.new(0, 10, 1, -16),
        BackgroundColor3 = PAL.elevated,
        ZIndex = 24,
    })
    AddCorner(trackBg, 3)

    local fill = New("Frame", trackBg, {
        Size = UDim2.new(0, 0, 1, 0),
        BackgroundColor3 = PAL.accent1,
        ZIndex = 25,
    })
    AddCorner(fill, 3)

    local knob = New("Frame", trackBg, {
        Size = UDim2.new(0, 12, 0, 12),
        AnchorPoint = Vector2.new(0.5, 0.5),
        Position = UDim2.new(0, 0, 0.5, 0),
        BackgroundColor3 = PAL.text,
        ZIndex = 26,
    })
    AddCorner(knob, 6)

    local function setVal(v)
        v = math.clamp(v, minV, maxV)
        Cfg[cfgKey] = v
        local t = (v - minV) / (maxV - minV)
        badgeLabel.Text = string.format(fmt or "%g", v)
        fill.Size = UDim2.new(t, 0, 1, 0)
        knob.Position = UDim2.new(t, 0, 0.5, 0)
        if onChange then onChange(v) end
    end
    setVal(Cfg[cfgKey] or minV)

    local dragging = false
    knob.InputBegan:Connect(function(inp)
        if inp.UserInputType == Enum.UserInputType.MouseButton1
            or inp.UserInputType == Enum.UserInputType.Touch then
            dragging = true
        end
    end)
    UserInputService.InputEnded:Connect(function(inp)
        if inp.UserInputType == Enum.UserInputType.MouseButton1
            or inp.UserInputType == Enum.UserInputType.Touch then
            dragging = false
        end
    end)
    UserInputService.InputChanged:Connect(function(inp)
        if dragging and (inp.UserInputType == Enum.UserInputType.MouseMovement
            or inp.UserInputType == Enum.UserInputType.Touch) then
            local abs, size = trackBg.AbsolutePosition, trackBg.AbsoluteSize
            if size.X > 0 then
                local t = math.clamp((inp.Position.X - abs.X) / size.X, 0, 1)
                setVal(minV + (maxV - minV) * t)
            end
        end
    end)

    return row, setVal
end

local function AddDropdown(page, label, cfgKey, options, onChange)
    widgetCounter = widgetCounter + 1
    local row = New("Frame", page, {
        Size = UDim2.new(1, 0, 0, 34),
        BackgroundColor3 = PAL.card,
        BackgroundTransparency = 0.5,
        ZIndex = 23,
        LayoutOrder = widgetCounter,
    })
    AddCorner(row, 7)

    New("TextLabel", row, {
        Size = UDim2.new(0.55, 0, 1, 0),
        Position = UDim2.new(0, 10, 0, 0),
        BackgroundTransparency = 1,
        Text = label,
        TextColor3 = PAL.text,
        TextSize = 12,
        Font = Enum.Font.Gotham,
        TextXAlignment = Enum.TextXAlignment.Left,
        ZIndex = 24,
    })

    local ddBtn = New("TextButton", row, {
        Size = UDim2.new(0, 140, 0, 22),
        Position = UDim2.new(1, -150, 0.5, -11),
        BackgroundColor3 = PAL.elevated,
        Text = tostring(Cfg[cfgKey] or ""),
        TextColor3 = PAL.text,
        TextSize = 11,
        Font = Enum.Font.Gotham,
        ZIndex = 25,
    })
    AddCorner(ddBtn, 6)

    local popup = nil

    ddBtn.MouseButton1Click:Connect(function()
        if popup then popup:Destroy() popup = nil return end

        local scale = Cfg.GuiScale
        local bp = ddBtn.AbsolutePosition
        local mp = menuGui.AbsolutePosition
        local px = (bp.X - mp.X) / scale
        local py = (bp.Y - mp.Y) / scale + 24

        popup = New("Frame", menuGui, {
            Size = UDim2.new(0, 140, 0, #options * 26 + 8),
            Position = UDim2.new(0, px, 0, py),
            BackgroundColor3 = PAL.elevated,
            ZIndex = 200,
        })
        AddCorner(popup, 6)
        AddStroke(popup, PAL.stroke, 1, 0.2)
        New("UIListLayout", popup, {
            Padding = UDim.new(0, 2),
            FillDirection = Enum.FillDirection.Vertical,
            HorizontalAlignment = Enum.HorizontalAlignment.Center,
        })
        New("UIPadding", popup, {
            PaddingTop = UDim.new(0, 4),
            PaddingBottom = UDim.new(0, 4),
        })

        for _, opt in ipairs(options) do
            local ob = New("TextButton", popup, {
                Size = UDim2.new(1, -8, 0, 22),
                BackgroundTransparency = 1,
                Text = opt,
                TextColor3 = PAL.text,
                TextSize = 11,
                Font = Enum.Font.Gotham,
                ZIndex = 201,
            })
            AddCorner(ob, 4)
            ob.MouseEnter:Connect(function()
                ob.BackgroundTransparency = 0.5
                ob.BackgroundColor3 = PAL.card
            end)
            ob.MouseLeave:Connect(function()
                ob.BackgroundTransparency = 1
            end)
            ob.MouseButton1Click:Connect(function()
                Cfg[cfgKey] = opt
                ddBtn.Text = opt
                if onChange then onChange(opt) end
                if popup then popup:Destroy() popup = nil end
            end)
        end

        local closeConn
        closeConn = UserInputService.InputBegan:Connect(function(inp)
            if inp.UserInputType == Enum.UserInputType.MouseButton1 then
                task.wait()
                if popup then popup:Destroy() popup = nil end
                closeConn:Disconnect()
            end
        end)
    end)

    return row, function(v)
        Cfg[cfgKey] = v
        ddBtn.Text = tostring(v)
    end
end

local function AddButton(page, label, onClick)
    widgetCounter = widgetCounter + 1
    local row = New("TextButton", page, {
        Size = UDim2.new(1, 0, 0, 34),
        BackgroundColor3 = PAL.card,
        BackgroundTransparency = 0.3,
        Text = label,
        TextColor3 = PAL.text,
        TextSize = 12,
        Font = Enum.Font.GothamBold,
        ZIndex = 23,
        LayoutOrder = widgetCounter,
    })
    AddCorner(row, 7)
    row.MouseEnter:Connect(function()
        Tween(row, TweenInfo.new(0.1), { BackgroundTransparency = 0.1 })
    end)
    row.MouseLeave:Connect(function()
        Tween(row, TweenInfo.new(0.1), { BackgroundTransparency = 0.3 })
    end)
    row.MouseButton1Click:Connect(function()
        if onClick then pcall(onClick) end
    end)
    return row
end

local function AddTextInput(page, label, cfgKey, placeholder, onCommit)
    widgetCounter = widgetCounter + 1
    local row = New("Frame", page, {
        Size = UDim2.new(1, 0, 0, 34),
        BackgroundColor3 = PAL.card,
        BackgroundTransparency = 0.5,
        ZIndex = 23,
        LayoutOrder = widgetCounter,
    })
    AddCorner(row, 7)

    New("TextLabel", row, {
        Size = UDim2.new(0.5, 0, 1, 0),
        Position = UDim2.new(0, 10, 0, 0),
        BackgroundTransparency = 1,
        Text = label,
        TextColor3 = PAL.text,
        TextSize = 12,
        Font = Enum.Font.Gotham,
        TextXAlignment = Enum.TextXAlignment.Left,
        ZIndex = 24,
    })

    local box = New("TextBox", row, {
        Size = UDim2.new(0, 150, 0, 22),
        Position = UDim2.new(1, -160, 0.5, -11),
        BackgroundColor3 = PAL.elevated,
        Text = tostring(Cfg[cfgKey] or ""),
        PlaceholderText = placeholder or "",
        TextColor3 = PAL.text,
        PlaceholderColor3 = PAL.muted,
        TextSize = 11,
        Font = Enum.Font.Gotham,
        ClearTextOnFocus = false,
        ZIndex = 25,
    })
    AddCorner(box, 6)

    box.FocusLost:Connect(function()
        Cfg[cfgKey] = box.Text
        if onCommit then onCommit(box.Text) end
    end)

    return row, box
end

-- ============================================================
-- NOTIFICATIONS
-- ============================================================
local notifStack = {}
local notifGui = menuGui

local function ReflowNotifs()
    for i, n in ipairs(notifStack) do
        Tween(n, TweenInfo.new(0.2), {
            Position = UDim2.new(1, -294, 0, 14 + (i - 1) * 62),
        })
    end
end

V.Notify = function(title, message, duration)
    duration = duration or 4
    local notif = New("Frame", notifGui, {
        Size = UDim2.new(0, 280, 0, 54),
        Position = UDim2.new(1, 10, 0, 14),
        BackgroundColor3 = PAL.panel,
        BackgroundTransparency = 0.1,
        ZIndex = 100,
    })
    AddCorner(notif, 8)
    AddStroke(notif, PAL.stroke, 1, 0.2)

    New("Frame", notif, {
        Size = UDim2.new(0, 3, 1, 0),
        BackgroundColor3 = PAL.accent1,
        BorderSizePixel = 0,
        ZIndex = 101,
    })
    New("TextLabel", notif, {
        Size = UDim2.new(1, -16, 0, 20),
        Position = UDim2.new(0, 12, 0, 6),
        BackgroundTransparency = 1,
        Text = title,
        TextColor3 = PAL.text,
        TextSize = 12,
        Font = Enum.Font.GothamBold,
        TextXAlignment = Enum.TextXAlignment.Left,
        ZIndex = 101,
    })
    New("TextLabel", notif, {
        Size = UDim2.new(1, -16, 0, 18),
        Position = UDim2.new(0, 12, 0, 28),
        BackgroundTransparency = 1,
        Text = message,
        TextColor3 = PAL.muted,
        TextSize = 11,
        Font = Enum.Font.Gotham,
        TextXAlignment = Enum.TextXAlignment.Left,
        ZIndex = 101,
    })

    table.insert(notifStack, notif)
    ReflowNotifs()

    task.delay(duration, function()
        if not notif.Parent then return end
        Tween(notif, TweenInfo.new(0.3), {
            Position = UDim2.new(1, 10, 0, notif.Position.Y.Offset),
        })
        task.wait(0.35)
        for i, n in ipairs(notifStack) do
            if n == notif then table.remove(notifStack, i) break end
        end
        if notif.Parent then notif:Destroy() end
        ReflowNotifs()
    end)
end

-- ============================================================
-- TAB DEFINITIONS
-- ============================================================
local order = 0
local function NextOrder() order = order + 1 return order end

-- ESP TAB
local pgEsp = MakeTab("ESP", NextOrder())
AddSection(pgEsp, "Enemies")
AddToggle(pgEsp, "Enable ESP", "EspEnable")
AddToggle(pgEsp, "Team Check", "EspTeamCheck")
AddSlider(pgEsp, "Max Distance", "EspMaxDist", 50, 5000, "%d")
AddSection(pgEsp, "Box")
AddToggle(pgEsp, "Box", "EspBox")
AddToggle(pgEsp, "Corner Style", "EspCorner")
AddToggle(pgEsp, "Fill", "EspFill")
AddSlider(pgEsp, "Corner Length", "EspCornerLen", 0.05, 0.5, "%.2f")
AddSection(pgEsp, "Info")
AddToggle(pgEsp, "Name", "EspNames")
AddToggle(pgEsp, "Display Name", "EspDisplayName")
AddToggle(pgEsp, "Health Bar", "EspHealthBar")
AddToggle(pgEsp, "Health Text", "EspHealthText")
AddToggle(pgEsp, "Distance", "EspDistance")
AddToggle(pgEsp, "Weapon", "EspWeapon")
AddSection(pgEsp, "Extra")
AddToggle(pgEsp, "Head Dot", "EspHeadDot")
AddToggle(pgEsp, "Skeleton", "EspSkeleton")
AddSlider(pgEsp, "Skeleton Thickness", "EspSkelThick", 1, 4, "%.1f")
AddToggle(pgEsp, "Tracers", "EspTracers")
AddDropdown(pgEsp, "Tracer Origin", "EspTracerOrigin", {"Bottom","Center","Top"})
AddToggle(pgEsp, "Chams", "EspChams")
AddToggle(pgEsp, "Off-screen Arrows", "EspOffscreen")
AddToggle(pgEsp, "Team Color", "EspTeamColor")
AddToggle(pgEsp, "Visibility Dot", "EspVisibleDot")

-- AIMBOT TAB
local pgAim = MakeTab("Aimbot", NextOrder())
AddSection(pgAim, "Targeting")
AddToggle(pgAim, "Enable Aimbot", "AimEnable")
AddToggle(pgAim, "Draw FOV Circle", "AimFovCircle")
AddSlider(pgAim, "FOV Radius", "AimFOV", 1, 500, "%d")
AddSlider(pgAim, "Smoothness", "AimSmooth", 0, 1, "%.2f")
AddSlider(pgAim, "Max Distance", "AimMaxDist", 100, 5000, "%d")
AddDropdown(pgAim, "Target Bone", "AimBone", {"Head","Torso","Nearest"})
AddDropdown(pgAim, "Aim Key", "AimKey", {"MouseButton2","MouseButton1","E","Q","LeftShift","Always"})
AddSection(pgAim, "Filters")
AddToggle(pgAim, "Wall Check", "AimWallCheck")
AddToggle(pgAim, "Team Check", "AimTeamCheck")
AddSection(pgAim, "Prediction")
AddToggle(pgAim, "Velocity Prediction", "AimPredict")
AddSlider(pgAim, "Prediction Scale", "AimPredScale", 0.2, 3.0, "%.2f")
AddSlider(pgAim, "Bullet Speed", "AimBulletSpeed", 100, 3000, "%d")

-- RAGE / HVH TAB
local pgRage = MakeTab("Rage", NextOrder())
AddSection(pgRage, "Rage Aimbot")
AddToggle(pgRage, "Enable Rage Aim", "RageAim")
AddSlider(pgRage, "Rage FOV", "RageFOV", 30, 360, "%d")
AddDropdown(pgRage, "Priority", "RagePriority", {"Distance","Health","FOV"})
AddToggle(pgRage, "Wall Penetration", "RageWallPen")
AddSection(pgRage, "Automation")
AddToggle(pgRage, "Auto Fire", "RageAutoFire")
AddToggle(pgRage, "Auto Reload", "RageAutoReload")
AddToggle(pgRage, "Auto Scope", "RageAutoScope")
AddToggle(pgRage, "Auto Respawn", "AutoRespawn")
AddSection(pgRage, "Safety")
AddToggle(pgRage, "Anti-Fling", "AntiFling")
AddToggle(pgRage, "Anti-Void", "AntiVoid")
AddSection(pgRage, "Rage Mode")
AddButton(pgRage, "Enable All Rage Features", function()
    Cfg.RageAim = true
    Cfg.RageAutoFire = true
    Cfg.RageAutoReload = true
    Cfg.RageWallPen = true
    Cfg.AntiFling = true
    Cfg.AntiVoid = true
    Cfg.AutoRespawn = true
    Notify("Rage", "All rage features enabled", 3)
end)
AddButton(pgRage, "Disable All Rage Features", function()
    Cfg.RageAim = false
    Cfg.RageAutoFire = false
    Cfg.RageAutoReload = false
    Cfg.RageAutoScope = false
    Notify("Rage", "All rage features disabled", 3)
end)

-- TELEPORT TAB
local pgTp = MakeTab("Teleport", NextOrder())
AddSection(pgTp, "Teleport To")
AddButton(pgTp, "Teleport to Mouse", function() Teleport.ToMouse() end)
AddButton(pgTp, "Save / Return Waypoint", function() Teleport.ToWaypoint() end)
AddSection(pgTp, "Teleport to Player")
AddDropdown(pgTp, "Target", "TpTarget", (function()
    local t = {}
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= LP then table.insert(t, p.Name) end
    end
    return t
end)())
AddButton(pgTp, "Teleport to Selected", function()
    local name = Cfg.TpTarget
    if not name then return Notify("Teleport", "No target selected", 2) end
    local target = Players:FindFirstChild(name)
    if not target then return Notify("Teleport", "Player not found", 2) end
    Teleport.ToPlayer(target)
end)

-- HITSOUNDS TAB
local pgHit = MakeTab("Hitsounds", NextOrder())
AddSection(pgHit, "Hitsounds")
AddToggle(pgHit, "Enable Hitsounds", "HitEnable")
AddDropdown(pgHit, "Style", "HitStyle", {"Rust","Skeet"})
AddSlider(pgHit, "Volume", "HitVolume", 0, 5, "%.2f")
AddToggle(pgHit, "Headshot Sound", "HitHeadshot")
AddToggle(pgHit, "Rage Hitsound", "HitRage")
AddSection(pgHit, "Assets")
AddTextInput(pgHit, "Rust Asset ID", "RustId", DEFAULT_HITSOUND)
AddTextInput(pgHit, "Skeet Asset ID", "SkeetId", DEFAULT_SKEET)

-- CROSSHAIR TAB
local pgCh = MakeTab("Crosshair", NextOrder())
AddSection(pgCh, "Crosshair")
AddToggle(pgCh, "Enable Crosshair", "ChEnable", function() ApplyCrosshair() end)
AddSlider(pgCh, "Size", "ChSize", 2, 40, "%d", function() ApplyCrosshair() end)
AddSlider(pgCh, "Thickness", "ChThick", 1, 6, "%d", function() ApplyCrosshair() end)
AddSlider(pgCh, "Gap", "ChGap", 0, 20, "%d", function() ApplyCrosshair() end)
AddToggle(pgCh, "Dot", "ChDot", function() ApplyCrosshair() end)
AddSection(pgCh, "Color")
AddSlider(pgCh, "Red", "ChR", 0, 255, "%d", function(v)
    Cfg.ChColor = {v, Cfg.ChColor[2], Cfg.ChColor[3]}
    ApplyCrosshair()
end)
AddSlider(pgCh, "Green", "ChG", 0, 255, "%d", function(v)
    Cfg.ChColor = {Cfg.ChColor[1], v, Cfg.ChColor[3]}
    ApplyCrosshair()
end)
AddSlider(pgCh, "Blue", "ChB", 0, 255, "%d", function(v)
    Cfg.ChColor = {Cfg.ChColor[1], Cfg.ChColor[2], v}
    ApplyCrosshair()
end)

-- VISUALS TAB
local pgVis = MakeTab("Visuals", NextOrder())
AddSection(pgVis, "World")
AddToggle(pgVis, "Fullbright", "Fullbright", function(v) ApplyFullbright(v) end)
AddToggle(pgVis, "No Fog", "NoFog", function(v) ApplyNoFog(v) end)
AddToggle(pgVis, "Low GFX", "LowGfx", function(v) ApplyLowGfx(v) end)
AddSection(pgVis, "Camera")
AddToggle(pgVis, "Custom FOV", "CustomFOV", function(v)
    Camera.FieldOfView = v and Cfg.FovValue or 70
end)
AddSlider(pgVis, "FOV Value", "FovValue", 60, 140, "%d", function(v)
    if Cfg.CustomFOV then Camera.FieldOfView = v end
end)
AddSection(pgVis, "HUD")
AddToggle(pgVis, "Spin Visual", "SpinVisual")
AddSlider(pgVis, "Spin Speed", "SpinSpeed", 10, 300, "%d")

-- RADAR TAB
local pgRadar = MakeTab("Radar", NextOrder())
AddSection(pgRadar, "Radar")
AddToggle(pgRadar, "Enable Radar", "RadarEnable")
AddSlider(pgRadar, "Radius", "RadarRadius", 60, 200, "%d")
AddSlider(pgRadar, "Scale", "RadarScale", 0.5, 10, "%.1f")
AddSlider(pgRadar, "Opacity", "RadarOpacity", 0.05, 0.95, "%.2f")

-- PLAYER LIST TAB
local pgPlist = MakeTab("Players", NextOrder())
AddSection(pgPlist, "Player List")
AddToggle(pgPlist, "Show Player List", "PlistEnable")
AddButton(pgPlist, "Refresh", function() UpdatePlayerList() end)

-- MOVEMENT TAB
local pgMove = MakeTab("Movement", NextOrder())
AddSection(pgMove, "Flight")
AddToggle(pgMove, "Fly", "FlyEnable", function(v) SetFly(v) end)
AddSlider(pgMove, "Fly Speed", "FlySpeed", 10, 300, "%d")
AddSection(pgMove, "Ground")
AddToggle(pgMove, "Speed Hack", "SpeedHack", function() UpdateSpeed() end)
AddSlider(pgMove, "Walk Speed", "WalkSpeed", 16, 200, "%d", function() UpdateSpeed() end)
AddToggle(pgMove, "Infinite Jump", "InfJump")
AddToggle(pgMove, "Noclip", "Noclip", function(v) SetNoclip(v) end)
AddSection(pgMove, "Utility")
AddToggle(pgMove, "Anti-AFK", "AntiAfk")

-- MISC TAB
local pgMisc = MakeTab("Misc", NextOrder())
AddSection(pgMisc, "Performance")
AddToggle(pgMisc, "Uncap FPS", "UncapFPS", function(v) setFpsCap(v and 0 or 60) end)
AddSection(pgMisc, "Server")
AddButton(pgMisc, "Rejoin", function()
    pcall(function() TeleportService:Teleport(game.PlaceId, LP) end)
end)
AddButton(pgMisc, "Server Hop", function()
    local ok, data = pcall(function()
        return TeleportService:GetPlayerPlaceInstanceAsync(LP.UserId)
    end)
    if ok and data then
        pcall(function()
            TeleportService:TeleportToPlaceInstance(game.PlaceId, data, LP)
        end)
    else
        pcall(function() TeleportService:Teleport(game.PlaceId, LP) end)
    end
end)

-- CONFIG TAB
local pgCfg = MakeTab("Config", NextOrder())
AddSection(pgCfg, "File")
AddButton(pgCfg, "Save Config", function()
    SaveCfg()
    Notify("Config", "Saved to " .. CFG_PATH, 3)
end)
AddButton(pgCfg, "Load Config", function()
    LoadCfg()
    Notify("Config", "Config loaded", 3)
end)
AddButton(pgCfg, "Reset All", function()
    for k, v in pairs(Cfg) do
        if type(v) == "boolean" then Cfg[k] = false end
    end
    Cfg.AntiAfk = true
    Cfg.GuiScale = 1.0
    Notify("Config", "All settings reset", 3)
end)
AddSection(pgCfg, "GUI")
AddSlider(pgCfg, "GUI Scale", "GuiScale", 0.7, 1.3, "%.2f", function(v)
    menuScale.Scale = v
end)

-- INFO TAB
local pgInfo = MakeTab("Info", NextOrder())
AddSection(pgInfo, "About")
do
    local about = New("Frame", pgInfo, {
        Size = UDim2.new(1, 0, 0, 80),
        BackgroundColor3 = PAL.card,
        BackgroundTransparency = 0.5,
        ZIndex = 23,
        LayoutOrder = widgetCounter + 1,
    })
    widgetCounter = widgetCounter + 1
    AddCorner(about, 8)
    New("TextLabel", about, {
        Size = UDim2.new(1, -16, 1, 0),
        Position = UDim2.new(0, 10, 0, 0),
        BackgroundTransparency = 1,
        Text = "Vodium v" .. SCRIPT_VERSION .. "\nGame: " .. game.Name
            .. "\nPlaceId: " .. tostring(game.PlaceId)
            .. "\nExecutor: " .. identifyExecutor(),
        TextColor3 = PAL.text,
        TextSize = 11,
        Font = Enum.Font.Gotham,
        TextXAlignment = Enum.TextXAlignment.Left,
        TextYAlignment = Enum.TextYAlignment.Top,
        ZIndex = 24,
        TextWrapped = true,
    })
end
AddSection(pgInfo, "Controls")
do
    local ctrl = New("Frame", pgInfo, {
        Size = UDim2.new(1, 0, 0, 90),
        BackgroundColor3 = PAL.card,
        BackgroundTransparency = 0.5,
        ZIndex = 23,
        LayoutOrder = widgetCounter + 1,
    })
    widgetCounter = widgetCounter + 1
    AddCorner(ctrl, 8)
    New("TextLabel", ctrl, {
        Size = UDim2.new(1, -16, 1, 0),
        Position = UDim2.new(0, 10, 0, 0),
        BackgroundTransparency = 1,
        Text = "LEFT ALT — Toggle Menu\nRIGHT MOUSE / Key — Aimbot\nWASD — Fly Direction\nSPACE / CTRL — Fly Up / Down\nR — Instant Respawn",
        TextColor3 = PAL.muted,
        TextSize = 11,
        Font = Enum.Font.Gotham,
        TextXAlignment = Enum.TextXAlignment.Left,
        TextYAlignment = Enum.TextYAlignment.Top,
        ZIndex = 24,
        TextWrapped = true,
    })
end

-- ============================================================
-- OPEN FIRST TAB
-- ============================================================
do
    local first = tabButtons["ESP"]
    if first then
        activeTab = "ESP"
        tabPages["ESP"].Visible = true
        first.btn.BackgroundColor3 = PAL.card
        first.btn.BackgroundTransparency = 0.5
        first.indicator.BackgroundTransparency = 0
    end
end

-- ============================================================
-- MENU VISIBILITY / DRAG / MINIMIZE
-- ============================================================
local menuVisible = true
local minimized = false

local function setMenuVisible(v)
    menuVisible = v
    mainPanel.Visible = v
    shadow.Visible = v
end

local function setMinimized(v)
    minimized = v
    if v then
        Tween(mainPanel, TweenInfo.new(0.2), { Size = UDim2.new(0, 780, 0, 52) })
        Tween(shadow, TweenInfo.new(0.2), { Size = UDim2.new(0, 804, 0, 76) })
    else
        Tween(mainPanel, TweenInfo.new(0.25), { Size = UDim2.new(0, 780, 0, 540) })
        Tween(shadow, TweenInfo.new(0.25), { Size = UDim2.new(0, 804, 0, 564) })
    end
end

closeBtn.MouseButton1Click:Connect(function() setMenuVisible(false) end)
minimizeBtn.MouseButton1Click:Connect(function() setMinimized(not minimized) end)

local dragging, dragStart, startPos = false, nil, nil
header.InputBegan:Connect(function(inp)
    if inp.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = inp.Position
        startPos = mainPanel.Position
    end
end)
UserInputService.InputEnded:Connect(function(inp)
    if inp.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
    end
end)
UserInputService.InputChanged:Connect(function(inp)
    if dragging and inp.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = inp.Position - dragStart
        local newPos = UDim2.new(
            startPos.X.Scale, startPos.X.Offset + delta.X,
            startPos.Y.Scale, startPos.Y.Offset + delta.Y
        )
        mainPanel.Position = newPos
        shadow.Position = UDim2.new(
            newPos.X.Scale, newPos.X.Offset - 12,
            newPos.Y.Scale, newPos.Y.Offset - 12
        )
    end
end)

-- Left Alt toggle
local prevAlt = false
Conn(RunService.Heartbeat, function()
    local a = UserInputService:IsKeyDown(Enum.KeyCode.LeftAlt)
        or UserInputService:IsKeyDown(Enum.KeyCode.RightAlt)
    if a and not prevAlt then
        if not UserInputService:GetFocusedTextBox() then
            setMenuVisible(not menuVisible)
        end
    end
    prevAlt = a
end)

-- ============================================================
-- FOV CIRCLE
-- ============================================================
local fovCircle = New("Frame", hudGui, {
    Size = UDim2.new(0, 2, 0, 2),
    BackgroundColor3 = PAL.accent1,
    BackgroundTransparency = 1,
    Visible = false,
    ZIndex = 5,
})
AddCorner(fovCircle, 9999)
AddStroke(fovCircle, PAL.accent1, 1.5, 0)

local function UpdateFovCircle()
    if not Cfg.AimFovCircle or not (Cfg.AimEnable or Cfg.RageAim) then
        fovCircle.Visible = false
        return
    end
    local r = Cfg.RageAim and Cfg.RageFOV or Cfg.AimFOV
    local ctr = ScreenCenter()
    fovCircle.Size = UDim2.new(0, r * 2, 0, r * 2)
    fovCircle.Position = UDim2.new(0, ctr.X - r, 0, ctr.Y - r)
    fovCircle.Visible = true
end

-- ============================================================
-- PLAYER SETUP
-- ============================================================
local function OnPlayerAdded(p)
    PInfoSetup(p)
    SetupEspPlayer(p)
    SetupHitsound(p)
    if p.Character then
        PInfoRefresh(p)
    end
end

for _, p in ipairs(Players:GetPlayers()) do
    OnPlayerAdded(p)
end

Conn(Players.PlayerAdded, OnPlayerAdded)
Conn(Players.PlayerRemoving, function(p)
    RemoveEntry(p)
    PInfo[p] = nil
    if plistRows[p] then
        plistRows[p].row:Destroy()
        plistRows[p] = nil
    end
    if radarPlayers[p] then
        radarPlayers[p]:Destroy()
        radarPlayers[p] = nil
    end
end)

-- Character trackers for every player
Conn(Players.PlayerAdded, function(p)
    if p == LP then return end
    local conn
    conn = p.CharacterAdded:Connect(function()
        task.wait(0.1)
        PInfoRefresh(p)
    end)
    table.insert(V.Connections, conn)
end)

-- Local player
Conn(LP.CharacterAdded, function()
    task.wait(0.2)
    UpdateSpeed()
    if Cfg.FlyEnable then SetFly(true) end
    if Cfg.SpeedHack then UpdateSpeed() end
    for _, e in pairs(Esp.cache) do
        if e.player == LP then RemoveEntry(LP) end
    end
end)

-- Damage indicator
Conn(LP.CharacterAdded, function(char)
    task.wait(0.5)
    local hum = char:FindFirstChildOfClass("Humanoid")
    if not hum then return end
    local lastHp = hum.Health
    local c = hum.HealthChanged:Connect(function(hp)
        if hp < lastHp then
            local root = char:FindFirstChild("HumanoidRootPart")
            if root then
                -- find nearest enemy to attribute
                local nearest, nd = nil, math.huge
                for _, p in ipairs(Players:GetPlayers()) do
                    if p ~= LP and not SameTeam(p) then
                        local pc = p.Character
                        local pr = pc and pc:FindFirstChild("HumanoidRootPart")
                        if pr then
                            local d = (pr.Position - root.Position).Magnitude
                            if d < nd then nd = d; nearest = p end
                        end
                    end
                end
                if nearest then
                    local nc = nearest.Character
                    local nr = nc and nc:FindFirstChild("HumanoidRootPart")
                    if nr then
                        ShowDamageIndicator(nr.Position)
                    end
                end
            end
        end
        lastHp = hp
    end)
    table.insert(V.Connections, c)
end)

-- ============================================================
-- MAIN LOOP
-- ============================================================
local frameCount, fpsAccum, fpsTimer = 0, 0, os.clock()
local lastStatusUpdate = 0

local function LoopMain(dt)
    -- ESP (throttled internally)
    pcall(RenderEsp, dt)

    -- ESP refs refresh (light)
    for _, p in ipairs(Players:GetPlayers()) do
        PInfoRefresh(p)
    end

    -- Aimbot
    pcall(RunAimbot, dt)
    pcall(RunAutoFire, dt)
    pcall(RunAutoReload)

    -- Misc checks
    pcall(CheckAntiVoid)
    pcall(CheckAntiFling, dt)

    -- Visuals
    pcall(UpdateFovCircle)
    pcall(UpdateFly)
    pcall(UpdateNoclip)
    pcall(UpdateDamageIndicators, dt)
    pcall(UpdateRadar)
    pcall(UpdateSpin, dt)

    -- Auto respawn
    if Cfg.AutoRespawn then
        local char = LP.Character
        local hum = char and char:FindFirstChildOfClass("Humanoid")
        if not char or (hum and hum.Health <= 0) then
            pcall(function() LP:LoadCharacter() end)
        end
    end

    -- Status bar
    frameCount = frameCount + 1
    local now = os.clock()
    if now - fpsTimer >= 1.0 then
        fpsAccum = frameCount / (now - fpsTimer)
        frameCount = 0
        fpsTimer = now
    end
    if now - lastStatusUpdate >= 0.5 then
        lastStatusUpdate = now
        local ping = 0
        pcall(function()
            if LP.GetNetworkPing then ping = math.floor(LP:GetNetworkPing() * 1000) end
        end)
        local t = os.date("*t")
        local clock = string.format("%02d:%02d", t.hour, t.min)
        statusLabel.Text = string.format("fps: %d · ping: %dms · %s · v%s",
            math.floor(fpsAccum), ping, clock, SCRIPT_VERSION)
    end
end

-- Player list refresh (throttled inside)
local plistAccum = 0
local radarAccum = 0
Conn(RunService.RenderStepped, function(dt)
    if V.Unloaded then return end
    pcall(LoopMain, dt)

    plistAccum = plistAccum + dt
    if plistAccum >= 0.3 then
        plistAccum = 0
        pcall(UpdatePlayerList)
    end

    radarAccum = radarAccum + dt
    if radarAccum >= 0.05 then
        radarAccum = 0
        pcall(UpdateRadar)
    end
end)

-- ============================================================
-- UNLOAD
-- ============================================================
function V:Unload()
    self.Unloaded = true

    pcall(function() SetFly(false) end)
    pcall(function() SetNoclip(false) end)
    pcall(function() ApplyFullbright(false) end)
    pcall(function() ApplyNoFog(false) end)
    pcall(function() ApplyLowGfx(false) end)

    pcall(function()
        Camera.FieldOfView = 70
        Camera.CameraType = Enum.CameraType.Custom
    end)

    local char = LP.Character
    if char then
        local hum = char:FindFirstChildOfClass("Humanoid")
        if hum then
            hum.WalkSpeed = 16
            hum.JumpPower = 50
            hum.AutoRotate = true
            hum.PlatformStand = false
        end
    end

    for player, _ in pairs(Esp.cache) do
        RemoveEntry(player)
    end

    for _, c in ipairs(self.Connections) do
        pcall(function() c:Disconnect() end)
    end
    for _, inst in ipairs(self.Instances) do
        pcall(function() inst:Destroy() end)
    end
    for _, fn in ipairs(self.ExtraCleanup) do
        pcall(fn)
    end

    if afkThread then pcall(task.cancel, afkThread) end

    pcall(SaveCfg)

    if getgenv then getgenv().VodiumLoaded = nil end
end

if getgenv then getgenv().VodiumLoaded = V end

-- ============================================================
-- INIT
-- ============================================================
LoadCfg()
ApplyCrosshair()
UpdateSpeed()
StartAntiAfk()

task.defer(function()
    Notify("Vodium v" .. SCRIPT_VERSION, "Loaded. LEFT ALT to toggle menu.", 6)
end)
