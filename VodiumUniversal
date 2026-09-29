--[[
    VODIUM  UI Library  —  Full Build v4
]]
local VALID_KEY      = "VODIUM-6J3M-X2QP-W5FE"
local KICK_MESSAGE   = "Fuck u NIGGA"
local LOAD_TIME      = 10
local TOGGLE_CLOSE   = Enum.KeyCode.LeftAlt

local LOGO_ASSET     = "118589829721348"
local CLICK_SOUND    = "rbxassetid://88442833509532"
local VALID_SOUND    = "rbxassetid://128842283247970"
local FALLBACK_CLICK = "rbxasset://sounds/electronicpingshort.wav"
local FALLBACK_VALID = "rbxasset://sounds/impact_water.mp3"

local LOGO_THUMB     = "rbxthumb://type=Asset&id=" .. LOGO_ASSET .. "&w=150&h=150"
local LOGO_RAW       = "rbxassetid://" .. LOGO_ASSET

local Players      = game:GetService("Players")
local UIS          = game:GetService("UserInputService")
local CAS          = game:GetService("ContextActionService")
local TweenService = game:GetService("TweenService")
local RunService   = game:GetService("RunService")
local SoundService = game:GetService("SoundService")
local Workspace    = game:GetService("Workspace")
local Lighting     = game:GetService("Lighting")
local LocalPlayer  = Players.LocalPlayer
local Camera       = Workspace.CurrentCamera

local C = {
    Void    = Color3.fromRGB(6, 6, 8),
    Bg      = Color3.fromRGB(10, 10, 13),
    Panel   = Color3.fromRGB(16, 16, 21),
    Side    = Color3.fromRGB(14, 14, 18),
    Raised  = Color3.fromRGB(24, 25, 32),
    Hover   = Color3.fromRGB(36, 38, 48),
    Line    = Color3.fromRGB(70, 78, 96),
    Steel   = Color3.fromRGB(198, 210, 228),
    Chrome  = Color3.fromRGB(240, 244, 252),
    Mute    = Color3.fromRGB(130, 138, 154),
    Text    = Color3.fromRGB(242, 244, 250),
    Dim     = Color3.fromRGB(160, 168, 182),
    Bad     = Color3.fromRGB(220, 78, 90),
    Good    = Color3.fromRGB(132, 204, 168),
    Accent  = Color3.fromRGB(108, 140, 255),
}

local function tw(obj, props, t, style, dir)
    local info = TweenInfo.new(t or 0.24, style or Enum.EasingStyle.Quint, dir or Enum.EasingDirection.Out)
    local twn = TweenService:Create(obj, info, props)
    twn:Play()
    return twn
end

local function corner(p, r)
    local c = Instance.new("UICorner")
    c.CornerRadius = UDim.new(0, r or 8)
    c.Parent = p
    return c
end

local function stroke(p, col, th, tr)
    local s = Instance.new("UIStroke")
    s.Color = col or C.Line
    s.Thickness = th or 1
    s.Transparency = tr == nil and 0.4 or tr
    s.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    s.Parent = p
    return s
end

local function gradient(p, a, b, rot)
    local g = Instance.new("UIGradient")
    g.Color = ColorSequence.new(a, b)
    g.Rotation = rot or 90
    g.Parent = p
    return g
end

local function pad(p, t, r, b, l)
    local x = Instance.new("UIPadding")
    x.PaddingTop    = UDim.new(0, t or 0)
    x.PaddingRight  = UDim.new(0, r or t or 0)
    x.PaddingBottom = UDim.new(0, b or t or 0)
    x.PaddingLeft   = UDim.new(0, l or t or 0)
    x.Parent = p
    return x
end

local function sgn(x) return x > 0 and 1 or (x < 0 and -1 or 0) end

local function getUiParent()
    local ok, hui = pcall(function()
        return (gethui and gethui()) or (get_hidden_gui and get_hidden_gui())
    end)
    if ok and hui then return hui end
    local cg = game:GetService("CoreGui")
    local ok2 = pcall(function()
        local t = Instance.new("Folder")
        t.Parent = cg
        t:Destroy()
    end)
    if ok2 then return cg end
    return LocalPlayer:WaitForChild("PlayerGui")
end

local PARENT = getUiParent()
pcall(function()
    local old = PARENT:FindFirstChild("VodiumLib")
    if old then old:Destroy() end
end)

local Gui = Instance.new("ScreenGui")
Gui.Name              = "VodiumLib"
Gui.ResetOnSpawn      = false
Gui.IgnoreGuiInset    = true
Gui.ZIndexBehavior    = Enum.ZIndexBehavior.Sibling
Gui.Parent            = PARENT

local function makeLogo(parent, size, z)
    local img = Instance.new("ImageLabel")
    img.BackgroundTransparency = 1
    img.Size  = typeof(size) == "UDim2" and size or UDim2.fromOffset(size, size)
    img.Image = LOGO_THUMB
    img.ScaleType = Enum.ScaleType.Fit
    img.ZIndex    = z or ((parent.ZIndex or 1) + 1)
    img.Parent    = parent
    task.delay(0.8, function()
        if img.Parent and img.IsLoaded == false then
            img.Image = LOGO_RAW
        end
    end)
    return img
end

-- ── sounds ──────────────────────────────────────────────────────────
local clickTpl = Instance.new("Sound")
clickTpl.Name    = "Click"
clickTpl.SoundId = CLICK_SOUND
clickTpl.Volume  = 1.0
clickTpl.Parent  = Gui

local validTpl = Instance.new("Sound")
validTpl.Name    = "Valid"
validTpl.SoundId = VALID_SOUND
validTpl.Volume  = 1.8
validTpl.Parent  = Gui

local function fire(tpl, fallback)
    local function tryPlay(id)
        local s = Instance.new("Sound")
        s.SoundId = id
        s.Volume  = tpl.Volume
        s.Parent  = Gui
        SoundService:PlayLocalSound(s)
        s:Destroy()
    end
    local ok = pcall(function()
        if tpl.TimeLength > 0 or tpl.IsLoaded then
            SoundService:PlayLocalSound(tpl)
        else
            tryPlay(tpl.SoundId)
        end
    end)
    if not ok then pcall(tryPlay, fallback) end
end

local function playClick() fire(clickTpl, FALLBACK_CLICK) end
local function playValid() fire(validTpl, FALLBACK_VALID) end

-- ── notification holder ─────────────────────────────────────────────
local NotifyHolder = Instance.new("Frame")
NotifyHolder.BackgroundTransparency = 1
NotifyHolder.AnchorPoint = Vector2.new(1, 1)
NotifyHolder.Position    = UDim2.new(1, -18, 1, -18)
NotifyHolder.Size        = UDim2.new(0, 320, 1, -36)
NotifyHolder.ZIndex      = 90
NotifyHolder.Parent      = Gui
local nLay = Instance.new("UIListLayout")
nLay.VerticalAlignment = Enum.VerticalAlignment.Bottom
nLay.Padding           = UDim.new(0, 8)
nLay.Parent            = NotifyHolder

local Library = {}

function Library:Notify(title, body, dur)
    dur = dur or 3
    local card = Instance.new("Frame")
    card.BackgroundColor3 = C.Panel
    card.Size             = UDim2.new(1, 40, 0, 0)
    card.ClipsDescendants = true
    card.ZIndex           = 91
    card.Parent           = NotifyHolder
    corner(card, 8)
    stroke(card, C.Steel, 1, 0.45)

    local accent = Instance.new("Frame")
    accent.BackgroundColor3 = C.Accent
    accent.BorderSizePixel  = 0
    accent.Size             = UDim2.fromOffset(3, 0)
    accent.Position         = UDim2.new(0, 0, 0, 0)
    accent.AutomaticSize    = Enum.AutomaticSize.Y
    accent.ZIndex           = 92
    accent.Parent           = card
    corner(accent, 2)

    local ic = makeLogo(card, 26, 92)
    ic.Position = UDim2.fromOffset(14, 10)

    local t = Instance.new("TextLabel")
    t.BackgroundTransparency = 1
    t.Position       = UDim2.fromOffset(48, 8)
    t.Size           = UDim2.new(1, -60, 0, 18)
    t.Font           = Enum.Font.GothamBold
    t.TextSize       = 13
    t.TextXAlignment = Enum.TextXAlignment.Left
    t.TextColor3     = C.Chrome
    t.Text           = title or "Vodium"
    t.ZIndex         = 92
    t.Parent         = card

    local b = Instance.new("TextLabel")
    b.BackgroundTransparency = 1
    b.Position       = UDim2.fromOffset(48, 26)
    b.Size           = UDim2.new(1, -60, 0, 32)
    b.Font           = Enum.Font.Gotham
    b.TextSize       = 12
    b.TextWrapped    = true
    b.TextXAlignment = Enum.TextXAlignment.Left
    b.TextYAlignment = Enum.TextYAlignment.Top
    b.TextColor3     = C.Dim
    b.Text           = body or ""
    b.ZIndex         = 92
    b.Parent         = card

    local pbTrack = Instance.new("Frame")
    pbTrack.BackgroundColor3 = C.Raised
    pbTrack.Position         = UDim2.new(0, 3, 1, -5)
    pbTrack.Size             = UDim2.new(1, -3, 0, 2)
    pbTrack.BorderSizePixel  = 0
    pbTrack.ZIndex           = 93
    pbTrack.Parent           = card

    local pbFill = Instance.new("Frame")
    pbFill.BackgroundColor3 = C.Accent
    pbFill.BorderSizePixel  = 0
    pbFill.Size             = UDim2.new(1, 0, 1, 0)
    pbFill.ZIndex           = 94
    pbFill.Parent           = pbTrack

    tw(card, {Size = UDim2.new(1, 0, 0, 72)}, 0.28)
    task.spawn(function()
        task.wait(0.3)
        tw(pbFill, {Size = UDim2.new(0, 0, 1, 0)}, dur - 0.1, Enum.EasingStyle.Linear)
        task.wait(dur)
        if not card.Parent then return end
        tw(card, {Size = UDim2.new(1, 30, 0, 0)}, 0.2)
        task.wait(0.22)
        card:Destroy()
    end)
end

local function hover(btn, rest, over)
    btn.MouseEnter:Connect(function() tw(btn, {BackgroundColor3 = over  or C.Hover},  0.12) end)
    btn.MouseLeave:Connect(function() tw(btn, {BackgroundColor3 = rest or C.Raised}, 0.12) end)
end

local function makeDrag(handle, target)
    local dragging, origin, startPos
    handle.InputBegan:Connect(function(input)
        if input.UserInputType ~= Enum.UserInputType.MouseButton1 then return end
        dragging  = true
        origin    = input.Position
        startPos  = target.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then dragging = false end
        end)
    end)
    UIS.InputChanged:Connect(function(input)
        if not dragging or input.UserInputType ~= Enum.UserInputType.MouseMovement then return end
        local d = input.Position - origin
        target.Position = UDim2.new(
            startPos.X.Scale, startPos.X.Offset + d.X,
            startPos.Y.Scale, startPos.Y.Offset + d.Y
        )
    end)
end

local function onClick(btn, fn)
    btn.MouseButton1Click:Connect(function()
        playClick()
        if fn then fn() end
    end)
end

local function bindMatch(bind, input)
    if typeof(bind) ~= "EnumItem" then return false end
    if bind.EnumType == Enum.KeyCode        then return input.KeyCode == bind end
    if bind.EnumType == Enum.UserInputType  then return input.UserInputType == bind end
    return false
end

local function isBindDown(bind)
    if typeof(bind) ~= "EnumItem" then return false end
    if bind.EnumType == Enum.KeyCode        then return UIS:IsKeyDown(bind) end
    if bind.EnumType == Enum.UserInputType  then return UIS:IsMouseButtonPressed(bind) end
    return false
end

-- =====================================================================
--  FEATURE STATE
-- =====================================================================
local F = {
    Esp = false, TeamCheck = false, Boxes = false, Fill = false,
    Distance = false, NameEsp = false, HpEsp = false, Skeletons = false,
    EspDistance = 1000,

    Aimbot = false, FovCircle = false, Fov = 120, HitboxExpander = false,
    HitboxSize = 12, AutoShoot = false, AimPart = "Head",
    AimbotTeamCheck = false, WallCheck = false, AimPrediction = 0,
    AimGravityComp = 0,
    AimMode = "Hold",
    AimKey = Enum.UserInputType.MouseButton2,
    PriorityMode = "Crosshair",

    -- advanced aim
    AimStyle       = "Smooth",
    AimHorizontal  = 35,
    AimVertical    = 35,
    AimHumanize    = 0,
    AimStick       = true,
    AimStickBoost  = 130,

    -- fire control
    AutoFireRate     = 10,
    AutoShootVisible = true,
    TriggerBot       = false,
    TriggerFov       = 3,
    TriggerVisible   = true,

    AntiAim = false,
    AAYawBase = "Local", AAYawOffset = 0, AAManualYaw = 0,
    AAPitchMode = "None", AAPitchAngle = 89,
    AARoll = false, AARollSpeed = 5,
    AAJitterRange = 90, AASpinSpeed = 14, AASpinYaw = 0,
    AAOnMove = false, AAOnJump = false, AAFirstPerson = false,

    ThirdPerson = false, TPOffset = 8, TPSens = 40, TPFaceCamera = true,

    Fullbright = false, NoFog = false, Vibrant = false,

    Fly = false, FlySpeed = 60, Speed = false, WalkSpeed = 16,
    JumpPower = false, JPValue = 50, InfJump = false, Noclip = false,
    Freecam = false, FC_Speed = 1,
    TpTarget = "",
}

local State = {
    OriginalLighting = {
        Brightness     = Lighting.Brightness,
        ClockTime      = Lighting.ClockTime,
        FogEnd         = Lighting.FogEnd,
        FogStart       = Lighting.FogStart,
        GlobalShadows  = Lighting.GlobalShadows,
        OutdoorAmbient = Lighting.OutdoorAmbient,
        Ambient        = Lighting.Ambient,
        EnvironmentDiffuseScale  = Lighting.EnvironmentDiffuseScale,
        EnvironmentSpecularScale = Lighting.EnvironmentSpecularScale,
    },
    OriginalAtmosphere = nil,
    OriginalWS = 16, OriginalJP = 50,
    OriginalNeckC0 = nil,
    FreecamPos = nil, FreecamVelocity = Vector3.new(),
    FreecamYaw = 0, FreecamPitch = 0,
    PreFreecamCameraType = nil,
    Connections = {}, EspDrawings = {}, FovDrawing = nil,
    HitboxCache = {}, AimToggled = false, LastShot = 0,
    AATick = 0, AASide = 1, AAWasActive = false,
    TPWasActive = false, TPYaw = 0, TPPitch = 0,
    TPInitialized = false,
    TPMouseBehaviorBackup = nil, TPCameraTypeBackup = nil, TPAutoRotateBackup = nil,
    Destroyed = false,
    ToggleOpen = nil,
    LastAltToggle = 0,

    -- aim internals
    LockedTarget      = nil,
    LockedTargetTime  = 0,
    UserMouseDelta    = 0,
    UserMouseTime     = 0,
    InjectedMouseTime = 0,

    -- adaptive aim calibration
    CalSensYaw   = -0.0035,
    CalSensPitch = -0.0035,
    LastCamLook  = nil,
    LastMouseSent = Vector2.new(0, 0),

    -- vibrant
    VibrantCC = nil,
}

do
    local atm = Lighting:FindFirstChildOfClass("Atmosphere")
    if atm then
        State.OriginalAtmosphere = {
            Density = atm.Density,
            Haze    = atm.Haze,
            Glare   = atm.Glare,
        }
    end
end

local function conn(name, c)
    State.Connections[name] = c
    return c
end

local function getChar(p)  return p and p.Character end
local function getRoot(p)
    local c = getChar(p)
    return c and c:FindFirstChild("HumanoidRootPart")
end
local function getHum(p)
    local c = getChar(p)
    return c and c:FindFirstChildOfClass("Humanoid")
end
local function isAlive(p)
    local h = getHum(p)
    return h and h.Health > 0
end
local function isTeammate(p)
    if p == LocalPlayer then return true end
    local lt, pt = LocalPlayer.Team, p.Team
    if lt and pt then return lt == pt end
    return false
end

-- =====================================================================
--  ESP
-- =====================================================================
local DRAWING_SUPPORTED = pcall(function()
    local t = Drawing.new("Square")
    if t and t.Remove then t:Remove() end
end)

local function destroyDrawings(d)
    if not d then return end
    for k, v in pairs(d) do
        if k == "Skel" and type(v) == "table" then
            for _, l in ipairs(v) do pcall(function() l:Remove() end) end
        else
            pcall(function() v:Remove() end)
        end
    end
end

local function clearEsp()
    for p, d in pairs(State.EspDrawings) do
        destroyDrawings(d)
        State.EspDrawings[p] = nil
    end
    table.clear(State.EspDrawings)
end

local function removePlayerEsp(p)
    local d = State.EspDrawings[p]
    if d then destroyDrawings(d); State.EspDrawings[p] = nil end
end

local function ensureDrawing(p)
    if not DRAWING_SUPPORTED then return nil end
    if State.EspDrawings[p] then return State.EspDrawings[p] end
    local d = {}
    d.Box = Drawing.new("Square")
    d.Box.Thickness = 1; d.Box.Filled = false; d.Box.Color = C.Chrome; d.Box.Visible = false

    d.FillBox = Drawing.new("Square")
    d.FillBox.Thickness = 1; d.FillBox.Filled = true; d.FillBox.Color = C.Steel
    d.FillBox.Transparency = 0.85; d.FillBox.Visible = false

    d.Name = Drawing.new("Text")
    d.Name.Size = 13; d.Name.Color = C.Chrome; d.Name.Center = true
    d.Name.Outline = true; d.Name.Visible = false

    d.Dist = Drawing.new("Text")
    d.Dist.Size = 12; d.Dist.Color = C.Dim; d.Dist.Center = true
    d.Dist.Outline = true; d.Dist.Visible = false

    d.HpBack = Drawing.new("Square")
    d.HpBack.Thickness = 1; d.HpBack.Filled = true
    d.HpBack.Color = Color3.new(0,0,0); d.HpBack.Visible = false

    d.HpFill = Drawing.new("Square")
    d.HpFill.Thickness = 1; d.HpFill.Filled = true
    d.HpFill.Color = C.Good; d.HpFill.Visible = false

    d.Skel = {}
    for i = 1, 15 do
        local l = Drawing.new("Line")
        l.Thickness = 1; l.Color = C.Steel; l.Visible = false
        d.Skel[i] = l
    end
    State.EspDrawings[p] = d
    return d
end

local BONE_MAP = {
    {"Head","UpperTorso"},{"UpperTorso","LowerTorso"},
    {"UpperTorso","LeftUpperArm"},{"LeftUpperArm","LeftLowerArm"},{"LeftLowerArm","LeftHand"},
    {"UpperTorso","RightUpperArm"},{"RightUpperArm","RightLowerArm"},{"RightLowerArm","RightHand"},
    {"LowerTorso","LeftUpperLeg"},{"LeftUpperLeg","LeftLowerLeg"},{"LeftLowerLeg","LeftFoot"},
    {"LowerTorso","RightUpperLeg"},{"RightUpperLeg","RightLowerLeg"},{"RightLowerLeg","RightFoot"},
}

local function w2s(pos)
    local ok, sp, onScreen = pcall(function() return Camera:WorldToViewportPoint(pos) end)
    if ok and onScreen and sp.Z > 0 then return sp.X, sp.Y, true end
    return 0, 0, false
end

local function hideDraw(d)
    if not d then return end
    for k, v in pairs(d) do
        if k == "Skel" and type(v) == "table" then
            for _, l in ipairs(v) do l.Visible = false end
        else
            pcall(function() v.Visible = false end)
        end
    end
end

local function updateEsp()
    if State.Destroyed then return end
    if not DRAWING_SUPPORTED then return end
    if not F.Esp then
        if next(State.EspDrawings) then clearEsp() end
        return
    end
    for _, p in pairs(Players:GetPlayers()) do
        if p == LocalPlayer then continue end
        local d = ensureDrawing(p)
        if not d then continue end

        local show = isAlive(p)
        if show and F.TeamCheck and isTeammate(p) then show = false end
        if not show then hideDraw(d) continue end

        local char = getChar(p)
        local root = char and char:FindFirstChild("HumanoidRootPart")
        local head = char and char:FindFirstChild("Head")
        if not root or not head then hideDraw(d) continue end

        local camPos   = Camera.CFrame.Position
        local distStuds = (camPos - root.Position).Magnitude
        if F.EspDistance > 0 and distStuds > F.EspDistance then hideDraw(d) continue end

        local headPos = head.Position + Vector3.new(0, 0.5, 0)
        local legPos  = root.Position - Vector3.new(0, 3, 0)
        local x1, y1, on1 = w2s(headPos)
        local x2, y2, on2 = w2s(legPos)
        if not (on1 and on2) then hideDraw(d) continue end

        local h   = math.abs(y2 - y1)
        local w   = h * 0.55
        local topX, topY = x1 - w/2, y1
        local dist = math.floor(distStuds)

        d.Box.Visible = F.Boxes
        if F.Boxes then
            d.Box.Position = Vector2.new(topX, topY)
            d.Box.Size     = Vector2.new(w, h)
        end
        d.FillBox.Visible = F.Fill
        if F.Fill then
            d.FillBox.Position = Vector2.new(topX, topY)
            d.FillBox.Size     = Vector2.new(w, h)
        end
        d.Name.Visible = F.NameEsp
        if F.NameEsp then
            d.Name.Text     = p.Name
            d.Name.Position = Vector2.new(x1, topY - 16)
        end
        d.Dist.Visible = F.Distance
        if F.Distance then
            d.Dist.Text     = tostring(dist) .. "m"
            d.Dist.Position = Vector2.new(x1, y2 + 4)
        end
        if F.HpEsp then
            local hum = getHum(p)
            local pct = hum and math.clamp(hum.Health / math.max(hum.MaxHealth, 1), 0, 1) or 0
            d.HpBack.Visible  = true
            d.HpBack.Position = Vector2.new(topX - 6, topY)
            d.HpBack.Size     = Vector2.new(3, h)
            d.HpFill.Visible  = true
            d.HpFill.Position = Vector2.new(topX - 6, topY + h * (1 - pct))
            d.HpFill.Size     = Vector2.new(3, h * pct)
            d.HpFill.Color    = Color3.fromRGB(
                math.clamp(220 * (1-pct) + 60, 0, 255),
                math.clamp(200 * pct  + 40, 0, 255),
                80
            )
        else
            d.HpBack.Visible = false; d.HpFill.Visible = false
        end
        if F.Skeletons then
            local pts = {}
            for _, pair in ipairs(BONE_MAP) do
                local a = char:FindFirstChild(pair[1], true)
                local b = char:FindFirstChild(pair[2], true)
                if a and b then
                    local ax, ay, ao = w2s(a.Position)
                    local bx, by, bo = w2s(b.Position)
                    if ao and bo then table.insert(pts, {ax, ay, bx, by}) end
                end
            end
            for i, l in ipairs(d.Skel) do
                local seg = pts[i]
                if seg then
                    l.Visible = true
                    l.From = Vector2.new(seg[1], seg[2])
                    l.To   = Vector2.new(seg[3], seg[4])
                else
                    l.Visible = false
                end
            end
        else
            for _, l in ipairs(d.Skel) do l.Visible = false end
        end
    end
end

Players.PlayerRemoving:Connect(function(p)
    removePlayerEsp(p)
    State.HitboxCache[p] = nil
    if State.LockedTarget == p then State.LockedTarget = nil end
end)

local function watchPlayer(p)
    if p == LocalPlayer then return end
    p.CharacterAdded:Connect(function() State.HitboxCache[p] = nil end)
end
for _, p in pairs(Players:GetPlayers()) do watchPlayer(p) end
Players.PlayerAdded:Connect(watchPlayer)

-- =====================================================================
--  COMBAT / AIMBOT  —  adaptive, universal
-- =====================================================================
local RAY_FILTER_TYPE = (Enum.RaycastFilterType and Enum.RaycastFilterType.Exclude)
    or (Enum.RaycastFilterType and Enum.RaycastFilterType.Blacklist)

local AIM_PART_ORDER = {
    Head             = {"Head", "UpperTorso", "Torso", "HumanoidRootPart"},
    HumanoidRootPart = {"HumanoidRootPart", "UpperTorso", "Torso", "Head"},
    Torso            = {"Torso", "UpperTorso", "LowerTorso", "HumanoidRootPart", "Head"},
    UpperTorso       = {"UpperTorso", "Torso", "HumanoidRootPart", "Head"},
}

local function getAimPart(p, name)
    local char = getChar(p)
    if not char then return nil end
    local order = AIM_PART_ORDER[name or F.AimPart] or {name, "Head", "HumanoidRootPart"}
    for _, partName in ipairs(order) do
        local part = char:FindFirstChild(partName)
        if part and part:IsA("BasePart") then return part end
    end
    for _, partName in ipairs(order) do
        local part = char:FindFirstChild(partName, true)
        if part and part:IsA("BasePart") then return part end
    end
    return nil
end

local function getPartVelocity(part)
    if not part then return Vector3.new() end
    local ok, v = pcall(function() return part.AssemblyLinearVelocity end)
    if ok and v then return v end
    return part.Velocity or Vector3.new()
end

local function getAimCenter()
    if UIS.MouseBehavior == Enum.MouseBehavior.LockCenter then
        local vp = Camera.ViewportSize
        return Vector2.new(vp.X * 0.5, vp.Y * 0.5)
    end
    return UIS:GetMouseLocation()
end

local function getScreenPos(worldPos)
    if not worldPos then return nil end
    local ok, sp, onScreen = pcall(function()
        return Camera:WorldToViewportPoint(worldPos)
    end)
    if not ok or not onScreen or sp.Z <= 0 then return nil end
    return Vector2.new(sp.X, sp.Y)
end

local function isVisible(targetPart)
    if not targetPart then return false end
    local char   = LocalPlayer.Character
    local origin = Camera.CFrame.Position
    local base   = targetPart.Position
    if (base - origin).Magnitude < 0.01 then return true end
    local params = RaycastParams.new()
    params.FilterType = RAY_FILTER_TYPE
    local filter = {}
    if char then table.insert(filter, char) end
    if targetPart.Parent then table.insert(filter, targetPart.Parent) end
    params.FilterDescendantsInstances = filter
    local samples = {
        base,
        base + Vector3.new(0, 1.5, 0),
        base - Vector3.new(0, 1.5, 0),
    }
    for _, pt in ipairs(samples) do
        local d = pt - origin
        if d.Magnitude > 0.01 then
            local ok, hit = pcall(function() return Workspace:Raycast(origin, d, params) end)
            if ok and hit == nil then return true end
        end
    end
    return false
end

local function getTargetFacingScore(p)
    local root  = getRoot(p)
    local lroot = getRoot(LocalPlayer)
    if not root or not lroot then return 0 end
    local toMe = (lroot.Position - root.Position)
    if toMe.Magnitude < 0.01 then return 0 end
    return root.CFrame.LookVector:Dot(toMe.Unit)
end

local function predictAimPos(part, root)
    local pos = part.Position
    if F.AimPrediction > 0 and root then
        pos = pos + getPartVelocity(root) * (F.AimPrediction / 100)
    end
    if F.AimGravityComp > 0 then
        pos = pos + Vector3.new(0, F.AimGravityComp, 0)
    end
    return pos
end

local function lookAngles(v)
    local yaw   = math.atan2(-v.X, -v.Z)
    local pitch = math.asin(math.clamp(v.Y, -1, 1))
    return yaw, pitch
end

local function angleDelta(a, b)
    local d = a - b
    while d >  math.pi do d = d - 2 * math.pi end
    while d < -math.pi do d = d + 2 * math.pi end
    return d
end

local function withinFov(worldPos, tolerance)
    local screen = getScreenPos(worldPos)
    if not screen then return nil end
    local d = (screen - getAimCenter()).Magnitude
    if d > F.Fov * (tolerance or 1) then return nil end
    return d
end

local function evaluateTarget(player, tolerance)
    if not player or not player.Parent then return nil end
    if player == LocalPlayer then return nil end
    if not isAlive(player) then return nil end
    if F.AimbotTeamCheck and isTeammate(player) then return nil end
    local part = getAimPart(player, F.AimPart)
    if not part then return nil end
    local root = getRoot(player)
    local aimPos = predictAimPos(part, root)
    local screenDist = withinFov(aimPos, tolerance)
    if not screenDist then return nil end
    if F.WallCheck and not isVisible(part) then return nil end
    local hum = getHum(player)
    return {
        player = player,
        part = part,
        aimPos = aimPos,
        screen = screenDist,
        dist = (Camera.CFrame.Position - part.Position).Magnitude,
        hp = hum and hum.Health or 0,
        facing = getTargetFacingScore(player),
    }
end

local function collectTargets()
    local list = {}
    for _, p in pairs(Players:GetPlayers()) do
        local rec = evaluateTarget(p, 1.0)
        if rec then table.insert(list, rec) end
    end
    return list
end

local function sortTargets(list, mode)
    if mode == "Closest" then
        table.sort(list, function(a,b) return a.dist < b.dist end)
    elseif mode == "LowestHP" then
        table.sort(list, function(a,b) return a.hp < b.hp end)
    elseif mode == "HighestHP" then
        table.sort(list, function(a,b) return a.hp > b.hp end)
    elseif mode == "Facing" then
        table.sort(list, function(a,b) return a.facing > b.facing end)
    else
        table.sort(list, function(a,b) return a.screen < b.screen end)
    end
end

local function selectTarget()
    if F.AimStick and State.LockedTarget and State.LockedTarget.Parent then
        local boost = math.max(1, (F.AimStickBoost or 130) / 100)
        local rec = evaluateTarget(State.LockedTarget, boost)
        if rec then
            State.LockedTargetTime = os.clock()
            return rec
        end
        State.LockedTarget = nil
    end
    local list = collectTargets()
    if #list == 0 then return nil end
    sortTargets(list, F.PriorityMode)
    local best = list[1]
    State.LockedTarget = best.player
    State.LockedTargetTime = os.clock()
    return best
end

local function drawFov()
    if State.Destroyed then return end
    if not State.FovDrawing then
        State.FovDrawing = Drawing.new("Circle")
        State.FovDrawing.Thickness = 1
        State.FovDrawing.Color     = C.Steel
        State.FovDrawing.Filled    = false
        State.FovDrawing.NumSides  = 64
    end
    local c = State.FovDrawing
    if F.FovCircle and F.Aimbot then
        c.Visible  = true
        c.Position = getAimCenter()
        c.Radius   = F.Fov
    else
        c.Visible = false
    end
end

conn("AimToggle", UIS.InputBegan:Connect(function(input, gpe)
    if State.Destroyed then return end
    if gpe then return end
    if not F.Aimbot then State.AimToggled = false; return end
    if F.AimMode ~= "Toggle" then return end
    if bindMatch(F.AimKey, input) then State.AimToggled = not State.AimToggled end
end))

local function aimbotActive()
    if not F.Aimbot then return false end
    if F.AimMode == "Always" then return true end
    if F.AimMode == "Hold"   then return isBindDown(F.AimKey) end
    if F.AimMode == "Toggle" then return State.AimToggled end
    return false
end

conn("UserMouseTrack", UIS.InputChanged:Connect(function(input)
    if State.Destroyed then return end
    if input.UserInputType ~= Enum.UserInputType.MouseMovement then return end
    if os.clock() - State.InjectedMouseTime < 0.02 then return end
    State.UserMouseDelta = Vector2.new(input.Delta.X, input.Delta.Y).Magnitude
    State.UserMouseTime  = os.clock()
end))

-- ── Adaptive sensitivity calibration ─────────────────────────────────
-- Measure how much the camera actually rotates per unit of injected
-- mouse movement. This lets us compensate for whatever gain the game
-- applies. Doesn't run when camera is Scriptable (mouse won't move it).
local function pushCalibration()
    local curLook = Camera.CFrame.LookVector
    if Camera.CameraType == Enum.CameraType.Scriptable then
        State.LastCamLook = curLook
        return
    end
    if State.LastCamLook and State.LastMouseSent.Magnitude > 1 then
        local curYaw, curPitch = lookAngles(curLook)
        local prevYaw, prevPitch = lookAngles(State.LastCamLook)
        local dYaw   = angleDelta(curYaw, prevYaw)
        local dPitch = curPitch - prevPitch
        if math.abs(State.LastMouseSent.X) > 1 then
            local est = dYaw / State.LastMouseSent.X
            if est == est and math.abs(est) > 0.0001 and math.abs(est) < 1 then
                State.CalSensYaw = State.CalSensYaw * 0.75 + est * 0.25
            end
        end
        if math.abs(State.LastMouseSent.Y) > 1 then
            local est = dPitch / State.LastMouseSent.Y
            if est == est and math.abs(est) > 0.0001 and math.abs(est) < 1 then
                State.CalSensPitch = State.CalSensPitch * 0.75 + est * 0.25
            end
        end
    end
    State.LastCamLook = curLook
end

local function applyAimDelta(targetWorldPos, dt)
    pushCalibration()

    local camPos = Camera.CFrame.Position
    local toTarget = targetWorldPos - camPos
    if toTarget.Magnitude < 0.01 then
        State.LastMouseSent = Vector2.new(0, 0)
        return
    end

    local targetScreen = getScreenPos(targetWorldPos)
    if not targetScreen then
        State.LastMouseSent = Vector2.new(0, 0)
        return
    end

    local center = getAimCenter()
    local delta = targetScreen - center
    if delta.Magnitude < 0.5 then
        State.LastMouseSent = Vector2.new(0, 0)
        return
    end

    local sh = math.clamp(F.AimHorizontal / 100, 0.01, 1)
    local sv = math.clamp(F.AimVertical   / 100, 0.01, 1)
    local factorX, factorY
    if F.AimStyle == "Snap" then
        factorX, factorY = 1, 1
    else
        factorX = math.clamp(1 - math.exp(-dt * sh * 45), 0, 1)
        factorY = math.clamp(1 - math.exp(-dt * sv * 45), 0, 1)
    end

    local humanize = 1
    if F.AimHumanize > 0 then
        local since = os.clock() - State.UserMouseTime
        if since < 0.12 then
            local intensity = math.clamp(State.UserMouseDelta / 25, 0, 1)
            humanize = 1 - intensity * (F.AimHumanize / 100)
        end
    end

    -- Angular error between camera and target
    local targetDir = toTarget.Unit
    local camLook   = Camera.CFrame.LookVector
    local camYaw, camPitch = lookAngles(camLook)
    local tgtYaw, tgtPitch = lookAngles(targetDir)
    local errYaw   = angleDelta(tgtYaw, camYaw)
    local errPitch = tgtPitch - camPitch

    -- Calibrated sensitivity (angle change per mouse pixel)
    local sYaw   = State.CalSensYaw
    local sPitch = State.CalSensPitch
    if math.abs(sYaw)   < 0.0001 then sYaw   = -0.0035 end
    if math.abs(sPitch) < 0.0001 then sPitch = -0.0035 end

    local mouseX = (errYaw   * factorX * humanize) / sYaw
    local mouseY = (errPitch * factorY * humanize) / sPitch

    local maxJump = 200
    if math.abs(mouseX) > maxJump then mouseX = sgn(mouseX) * maxJump end
    if math.abs(mouseY) > maxJump then mouseY = sgn(mouseY) * maxJump end

    local injected = false
    if type(mousemoverel) == "function" then
        local ok = pcall(mousemoverel, mouseX, mouseY)
        if ok then
            injected = true
            State.InjectedMouseTime = os.clock()
        end
    end

    -- Fallback: direct camera write. Only when we couldn't feed the
    -- mouse pipeline, or the camera is Scriptable (game doesn't drive).
    if not injected or Camera.CameraType == Enum.CameraType.Scriptable then
        local ok, desired = pcall(function() return CFrame.lookAt(camPos, targetWorldPos) end)
        if ok and desired then
            local lerp = math.min(factorX, factorY)
            Camera.CFrame = Camera.CFrame:Lerp(desired, lerp)
        end
        State.LastMouseSent = Vector2.new(0, 0)
    else
        State.LastMouseSent = Vector2.new(mouseX, mouseY)
    end
end

local function tryAutoShoot()
    if type(mouse1click) == "function" then
        pcall(mouse1click)
        return
    end
    if type(mouse1press) == "function" and type(mouse1release) == "function" then
        pcall(mouse1press)
        task.wait(0.02)
        pcall(mouse1release)
    end
end

local function shouldFire(target)
    if F.TriggerBot then
        local screen = getScreenPos(target.aimPos or target.part.Position)
        if not screen then return false end
        local dist = (screen - getAimCenter()).Magnitude
        if dist > F.TriggerFov then return false end
        if F.TriggerVisible and not isVisible(target.part) then return false end
        return true
    end
    if F.AutoShoot then
        if F.AutoShootVisible and not isVisible(target.part) then return false end
        return true
    end
    return false
end

local AIMBOT_BIND_NAME = "VodiumAimbotCam"
RunService:BindToRenderStep(AIMBOT_BIND_NAME, Enum.RenderPriority.Camera.Value + 2, function(dt)
    if State.Destroyed then return end
    if not aimbotActive() then return end
    local target = selectTarget()
    if not target then return end

    applyAimDelta(target.aimPos, dt)

    if shouldFire(target) then
        local now      = os.clock()
        local interval = 1 / math.max(F.AutoFireRate, 1)
        if now - State.LastShot > interval then
            State.LastShot = now
            task.spawn(tryAutoShoot)
        end
    end
end)

local function hitboxTick()
    if State.Destroyed then return end
    for _, p in pairs(Players:GetPlayers()) do
        if p == LocalPlayer then continue end
        local root = getRoot(p)
        if root then
            if F.HitboxExpander then
                if not State.HitboxCache[p] then
                    State.HitboxCache[p] = {Size = root.Size, CanCollide = root.CanCollide, Transparency = root.Transparency}
                end
                root.Size         = Vector3.new(F.HitboxSize, F.HitboxSize, F.HitboxSize)
                root.CanCollide   = false
                root.Transparency = 0.7
            elseif State.HitboxCache[p] then
                local orig = State.HitboxCache[p]
                root.Size         = orig.Size
                root.CanCollide   = orig.CanCollide
                root.Transparency = orig.Transparency
                State.HitboxCache[p] = nil
            end
        end
    end
end

-- =====================================================================
--  ANTI-AIM
-- =====================================================================
local function aaShouldRun()
    if not F.AntiAim then return false end
    local char = LocalPlayer.Character
    local hum  = char and char:FindFirstChildOfClass("Humanoid")
    if not hum then return false end
    if F.AAOnMove and hum.MoveDirection.Magnitude < 0.05 then return false end
    if F.AAOnJump then
        local st = hum:GetState()
        if st ~= Enum.HumanoidStateType.Jumping
            and st ~= Enum.HumanoidStateType.Freefall
            and st ~= Enum.HumanoidStateType.Landed then
            return false
        end
    end
    return true
end

local function aaCleanup()
    if not State.AAWasActive then return end
    State.AAWasActive = false
    local char = LocalPlayer.Character
    local hum  = char and char:FindFirstChildOfClass("Humanoid")
    if hum then pcall(function() hum.AutoRotate = true end) end
    local head = char and char:FindFirstChild("Head")
    local neck = head and head:FindFirstChild("Neck")
    if not neck and char then neck = char:FindFirstChild("Neck", true) end
    if neck and neck:IsA("Motor6D") and State.OriginalNeckC0 then
        pcall(function() neck.C0 = State.OriginalNeckC0 end)
        State.OriginalNeckC0 = nil
    end
end

local function aaComputeYaw()
    local camYaw = math.atan2(-Camera.CFrame.LookVector.X, -Camera.CFrame.LookVector.Z)
    local base   = F.AAYawBase
    if base == "Local"    then return camYaw
    elseif base == "Reverse"  then return camYaw + math.pi
    elseif base == "Manual"   then return math.rad(F.AAManualYaw)
    elseif base == "Spin"     then return math.rad(F.AASpinYaw)
    elseif base == "Jitter"   then return camYaw + math.rad(F.AAJitterRange * State.AASide)
    elseif base == "Random"   then return math.rad(math.random(-180, 180))
    elseif base == "Sideways" then return camYaw + math.rad(90 * State.AASide)
    end
    return camYaw
end

local function aaComputePitch()
    local mode = F.AAPitchMode
    if mode == "Down"   then return math.rad(F.AAPitchAngle)
    elseif mode == "Up"     then return math.rad(-F.AAPitchAngle)
    elseif mode == "Jitter" then return math.rad(F.AAPitchAngle * State.AASide)
    elseif mode == "Random" then return math.rad(math.random(-F.AAPitchAngle, F.AAPitchAngle))
    end
    return 0
end

local function antiAimTick(dt)
    if State.Destroyed then return end
    if not aaShouldRun() then aaCleanup(); return end
    local char = LocalPlayer.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")
    local hum  = char and char:FindFirstChildOfClass("Humanoid")
    if not root or not hum then return end
    if hum.AutoRotate then pcall(function() hum.AutoRotate = false end) end
    State.AAWasActive = true
    State.AATick = State.AATick + dt
    if F.AAYawBase == "Spin" then
        F.AASpinYaw = (F.AASpinYaw + F.AASpinSpeed * dt * 60) % 360
    end
    if F.AAYawBase == "Jitter" or F.AAYawBase == "Sideways" or F.AAPitchMode == "Jitter" then
        if State.AATick >= 1/30 then
            State.AATick = 0
            State.AASide = -State.AASide
        end
    end
    local yaw   = aaComputeYaw()   + math.rad(F.AAYawOffset)
    local pitch = aaComputePitch()
    local roll  = F.AARoll and math.rad((tick() * F.AARollSpeed * 60) % 360) or 0
    root.CFrame = CFrame.new(root.Position) * CFrame.fromOrientation(pitch, yaw, roll)
    pcall(function() root.AssemblyAngularVelocity = Vector3.new(0,0,0) end)
    if F.AAFirstPerson then
        local head = char:FindFirstChild("Head")
        local neck = head and head:FindFirstChild("Neck")
        if not neck then neck = char:FindFirstChild("Neck", true) end
        if neck and neck:IsA("Motor6D") then
            if not State.OriginalNeckC0 then State.OriginalNeckC0 = neck.C0 end
            neck.C0 = State.OriginalNeckC0
        end
        if head then
            pcall(function()
                head.CFrame = CFrame.new(head.Position) * CFrame.fromOrientation(pitch, yaw, roll)
            end)
        end
    end
end

-- =====================================================================
--  THIRD PERSON
-- =====================================================================
conn("TPLook", UIS.InputChanged:Connect(function(input)
    if State.Destroyed then return end
    if not F.ThirdPerson or F.Freecam then return end
    if input.UserInputType == Enum.UserInputType.MouseMovement then
        local sens = (F.TPSens / 100) * 0.6
        State.TPYaw   = State.TPYaw   - math.rad(input.Delta.X) * sens
        State.TPPitch = math.clamp(
            State.TPPitch - math.rad(input.Delta.Y) * sens,
            -math.rad(85), math.rad(85)
        )
    end
end))

local function restoreTP()
    if not State.TPWasActive then return end
    State.TPWasActive   = false
    State.TPInitialized = false
    pcall(function() Camera.CameraType = State.TPCameraTypeBackup or Enum.CameraType.Custom end)
    pcall(function() UIS.MouseBehavior = State.TPMouseBehaviorBackup or Enum.MouseBehavior.Default end)
    pcall(function()
        local char = LocalPlayer.Character
        if char then
            for _, d in pairs(char:GetDescendants()) do
                if d:IsA("BasePart") then d.LocalTransparencyModifier = 0 end
            end
            local hum = char:FindFirstChildOfClass("Humanoid")
            if hum then hum.AutoRotate = (State.TPAutoRotateBackup ~= false) end
        end
    end)
    State.TPCameraTypeBackup    = nil
    State.TPMouseBehaviorBackup = nil
    State.TPAutoRotateBackup    = nil
end

local TP_BIND_NAME = "VodiumThirdPerson"
RunService:BindToRenderStep(TP_BIND_NAME, Enum.RenderPriority.Camera.Value + 1, function(dt)
    if State.Destroyed then return end
    if not F.ThirdPerson or F.Freecam then restoreTP(); return end
    local char = LocalPlayer.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")
    local head = char and char:FindFirstChild("Head")
    local hum  = char and char:FindFirstChildOfClass("Humanoid")
    if not root or not head or not hum then restoreTP(); return end
    if not State.TPInitialized then
        State.TPCameraTypeBackup    = Camera.CameraType
        State.TPMouseBehaviorBackup = UIS.MouseBehavior
        State.TPAutoRotateBackup    = hum.AutoRotate
        local look    = Camera.CFrame.LookVector
        State.TPYaw   = math.atan2(-look.X, -look.Z)
        State.TPPitch = math.asin(math.clamp(look.Y, -1, 1))
        State.TPInitialized = true
    end
    if Camera.CameraType ~= Enum.CameraType.Scriptable then
        Camera.CameraType = Enum.CameraType.Scriptable
    end
    pcall(function()
        if UIS.MouseBehavior ~= Enum.MouseBehavior.LockCenter then
            UIS.MouseBehavior = Enum.MouseBehavior.LockCenter
        end
    end)
    if LocalPlayer.CameraMode ~= Enum.CameraMode.Classic then
        pcall(function() LocalPlayer.CameraMode = Enum.CameraMode.Classic end)
    end
    if aimbotActive() then
        local look    = Camera.CFrame.LookVector
        State.TPYaw   = math.atan2(-look.X, -look.Z)
        State.TPPitch = math.asin(math.clamp(look.Y, -1, 1))
    end
    for _, d in pairs(char:GetDescendants()) do
        if d:IsA("BasePart") and d.LocalTransparencyModifier ~= 0 then
            d.LocalTransparencyModifier = 0
        end
    end
    if F.TPFaceCamera and not F.AntiAim then
        pcall(function()
            root.CFrame = CFrame.new(root.Position) * CFrame.Angles(0, State.TPYaw, 0)
            root.AssemblyAngularVelocity = Vector3.new(0,0,0)
        end)
    end
    local lookAt = root.Position + Vector3.new(0, 1.8, 0)
    local cp = math.cos(State.TPPitch)
    local lookDir = Vector3.new(
        -math.sin(State.TPYaw) * cp,
         math.sin(State.TPPitch),
        -math.cos(State.TPYaw) * cp
    )
    local idealCamPos = lookAt - lookDir * F.TPOffset
    local finalCamPos = idealCamPos
    local rayVec = idealCamPos - lookAt
    if rayVec.Magnitude > 0.01 then
        local params = RaycastParams.new()
        params.FilterType                 = RAY_FILTER_TYPE
        params.FilterDescendantsInstances = { char }
        local hit = Workspace:Raycast(lookAt, rayVec, params)
        if hit then finalCamPos = hit.Position + rayVec.Unit * 0.4 end
    end
    Camera.CFrame   = CFrame.lookAt(finalCamPos, finalCamPos + lookDir)
    State.TPWasActive = true
end)

-- =====================================================================
--  VISUALS  (Fullbright / NoFog / Vibrant)
-- =====================================================================
local function getAtmosphere()
    return Lighting:FindFirstChildOfClass("Atmosphere")
end

local function ensureVibrantCC()
    if State.VibrantCC and State.VibrantCC.Parent then return State.VibrantCC end
    local cc = Lighting:FindFirstChild("VodiumVibrantCC")
    if not cc then
        cc = Instance.new("ColorCorrectionEffect")
        cc.Name = "VodiumVibrantCC"
        cc.Parent = Lighting
    end
    State.VibrantCC = cc
    return cc
end

local function visualsTick()
    if State.Destroyed then return end
    local o = State.OriginalLighting

    if F.Fullbright then
        Lighting.Brightness     = 3
        Lighting.ClockTime      = 12
        Lighting.GlobalShadows  = false
        Lighting.OutdoorAmbient = Color3.fromRGB(178, 178, 178)
        Lighting.Ambient        = Color3.fromRGB(178, 178, 178)
        pcall(function()
            Lighting.EnvironmentDiffuseScale  = 1
            Lighting.EnvironmentSpecularScale = 1
        end)
    else
        Lighting.Brightness     = o.Brightness
        Lighting.ClockTime      = o.ClockTime
        Lighting.GlobalShadows  = o.GlobalShadows
        Lighting.OutdoorAmbient = o.OutdoorAmbient
        Lighting.Ambient        = o.Ambient
        pcall(function()
            if o.EnvironmentDiffuseScale then
                Lighting.EnvironmentDiffuseScale  = o.EnvironmentDiffuseScale
            end
            if o.EnvironmentSpecularScale then
                Lighting.EnvironmentSpecularScale = o.EnvironmentSpecularScale
            end
        end)
    end

    if F.NoFog then
        Lighting.FogEnd   = 9e9
        Lighting.FogStart = 9e9
        local atm = getAtmosphere()
        if atm then
            atm.Density = 0
            atm.Haze    = 0
        end
    else
        Lighting.FogEnd   = o.FogEnd
        Lighting.FogStart = o.FogStart
        local atm = getAtmosphere()
        if atm and State.OriginalAtmosphere then
            atm.Density = State.OriginalAtmosphere.Density
            atm.Haze    = State.OriginalAtmosphere.Haze
        end
    end

    if F.Vibrant then
        local cc = ensureVibrantCC()
        cc.Enabled    = true
        cc.Saturation = 0.55
        cc.Contrast   = 0.12
        cc.Brightness = 0.05
        cc.TintColor  = Color3.fromRGB(255, 250, 245)
    elseif State.VibrantCC then
        State.VibrantCC.Enabled = false
    end
end

-- =====================================================================
--  MOVEMENT
-- =====================================================================
local function movementTick(dt)
    if State.Destroyed then return end
    local char = LocalPlayer.Character
    local hum  = char and char:FindFirstChildOfClass("Humanoid")
    local root = char and char:FindFirstChild("HumanoidRootPart")
    if hum then
        hum.WalkSpeed = F.Speed      and F.WalkSpeed or State.OriginalWS
        hum.JumpPower = F.JumpPower  and F.JPValue   or State.OriginalJP
    end
    if root then
        if F.Noclip then
            for _, v in pairs(char:GetDescendants()) do
                if v:IsA("BasePart") then v.CanCollide = false end
            end
        end
        if F.Fly then
            local dir = Vector3.new()
            if UIS:IsKeyDown(Enum.KeyCode.W) then dir += Camera.CFrame.LookVector end
            if UIS:IsKeyDown(Enum.KeyCode.S) then dir -= Camera.CFrame.LookVector end
            if UIS:IsKeyDown(Enum.KeyCode.D) then dir += Camera.CFrame.RightVector end
            if UIS:IsKeyDown(Enum.KeyCode.A) then dir -= Camera.CFrame.RightVector end
            if UIS:IsKeyDown(Enum.KeyCode.Space)       then dir += Vector3.new(0,1,0) end
            if UIS:IsKeyDown(Enum.KeyCode.LeftControl) then dir -= Vector3.new(0,1,0) end
            root.Velocity = dir * F.FlySpeed
        end
    end
end

conn("InfJump", UIS.JumpRequest:Connect(function()
    if State.Destroyed then return end
    if F.InfJump then
        local hum = getHum(LocalPlayer)
        if hum then hum:ChangeState(Enum.HumanoidStateType.Jumping) end
    end
end))

-- =====================================================================
--  FREECAM
-- =====================================================================
local function freecamTick(dt)
    if State.Destroyed then return end
    if not F.Freecam then
        if State.FreecamPos then
            Camera.CameraType = State.PreFreecamCameraType or Enum.CameraType.Custom
            pcall(function() UIS.MouseBehavior = Enum.MouseBehavior.Default end)
            State.FreecamPos          = nil
            State.FreecamVelocity     = Vector3.new()
            State.PreFreecamCameraType = nil
        end
        return
    end
    if not State.FreecamPos then
        State.FreecamPos           = Camera.CFrame.Position
        State.FreecamVelocity      = Vector3.new()
        local _, y, x              = Camera.CFrame:ToEulerAnglesYXZ()
        State.FreecamYaw           = y
        State.FreecamPitch         = x
        State.PreFreecamCameraType = Camera.CameraType
        pcall(function() UIS.MouseBehavior = Enum.MouseBehavior.LockCenter end)
    end
    Camera.CameraType = Enum.CameraType.Scriptable
    pcall(function()
        if UIS.MouseBehavior ~= Enum.MouseBehavior.LockCenter then
            UIS.MouseBehavior = Enum.MouseBehavior.LockCenter
        end
    end)
    local boost = UIS:IsKeyDown(Enum.KeyCode.LeftShift) and 4 or 1
    local slow  = UIS:IsKeyDown(Enum.KeyCode.LeftAlt) and 0.25 or 1
    local speed = F.FC_Speed * 60 * boost * slow
    local input = Vector3.new()
    if UIS:IsKeyDown(Enum.KeyCode.W) then input += Camera.CFrame.LookVector end
    if UIS:IsKeyDown(Enum.KeyCode.S) then input -= Camera.CFrame.LookVector end
    if UIS:IsKeyDown(Enum.KeyCode.D) then input += Camera.CFrame.RightVector end
    if UIS:IsKeyDown(Enum.KeyCode.A) then input -= Camera.CFrame.RightVector end
    if UIS:IsKeyDown(Enum.KeyCode.Space)       then input += Vector3.new(0,1,0) end
    if UIS:IsKeyDown(Enum.KeyCode.LeftControl) then input -= Vector3.new(0,1,0) end
    if input.Magnitude > 0.01 then input = input.Unit * speed else input = Vector3.new() end
    local accel            = 1 - math.exp(-dt * 14)
    State.FreecamVelocity  = State.FreecamVelocity:Lerp(input, accel)
    State.FreecamPos       = State.FreecamPos + State.FreecamVelocity * dt
    Camera.CFrame = CFrame.new(State.FreecamPos)
        * CFrame.Angles(0, State.FreecamYaw, 0)
        * CFrame.Angles(State.FreecamPitch, 0, 0)
end

conn("FreecamLook", UIS.InputChanged:Connect(function(input)
    if State.Destroyed then return end
    if F.Freecam and input.UserInputType == Enum.UserInputType.MouseMovement then
        State.FreecamYaw   = State.FreecamYaw - math.rad(input.Delta.X) * 0.4
        State.FreecamPitch = math.clamp(
            State.FreecamPitch - math.rad(input.Delta.Y) * 0.4,
            -math.pi/2 + 0.02, math.pi/2 - 0.02
        )
    end
end))

-- =====================================================================
--  MAIN LOOP
-- =====================================================================
conn("Main", RunService.RenderStepped:Connect(function(dt)
    if State.Destroyed then return end
    updateEsp(); drawFov(); hitboxTick(); visualsTick(); movementTick(dt); freecamTick(dt)
end))

conn("AA", RunService.Stepped:Connect(function(_, dt)
    if State.Destroyed then return end
    antiAimTick(dt)
end))

LocalPlayer.CharacterAdded:Connect(function(char)
    task.wait(1)
    local hum = char:FindFirstChildOfClass("Humanoid")
    if hum then
        State.OriginalWS = hum.WalkSpeed
        State.OriginalJP = hum.JumpPower
    end
    State.OriginalNeckC0 = nil
end)

-- =====================================================================
--  UI WIDGETS
-- =====================================================================
local function makeWidgets(parent)
    local Api = {}

    local function row(h)
        local r = Instance.new("Frame")
        r.BackgroundColor3 = C.Raised
        r.Size             = UDim2.new(1, 0, 0, h or 40)
        r.ZIndex           = (parent.ZIndex or 8) + 1
        r.Parent           = parent
        corner(r, 8)
        stroke(r, C.Line, 1, 0.55)
        return r
    end

    function Api:AddLabel(text)
        local l = Instance.new("TextLabel")
        l.BackgroundTransparency = 1
        l.Size           = UDim2.new(1, 0, 0, 18)
        l.Font           = Enum.Font.Gotham
        l.TextSize       = 13
        l.TextXAlignment = Enum.TextXAlignment.Left
        l.TextColor3     = C.Dim
        l.Text           = text or ""
        l.ZIndex         = (parent.ZIndex or 8) + 1
        l.Parent         = parent
        return l
    end

    function Api:AddButton(cfg)
        cfg = cfg or {}
        local b = Instance.new("TextButton")
        b.BackgroundColor3 = C.Raised
        b.Size             = UDim2.new(1, 0, 0, 38)
        b.Font             = Enum.Font.GothamMedium
        b.TextSize         = 14
        b.TextColor3       = C.Text
        b.Text             = cfg.Name or "Button"
        b.AutoButtonColor  = false
        b.ZIndex           = (parent.ZIndex or 8) + 1
        b.Parent           = parent
        corner(b, 8)
        stroke(b, C.Line, 1, 0.4)
        hover(b, C.Raised, C.Hover)
        onClick(b, cfg.Callback)
        return b
    end

    function Api:AddToggle(cfg)
        cfg = cfg or {}
        local r = row(40)
        local l = Instance.new("TextLabel")
        l.BackgroundTransparency = 1
        l.Position       = UDim2.fromOffset(14, 0)
        l.Size           = UDim2.new(1, -70, 1, 0)
        l.Font           = Enum.Font.Gotham
        l.TextSize       = 14
        l.TextXAlignment = Enum.TextXAlignment.Left
        l.TextColor3     = C.Text
        l.Text           = cfg.Name or "Toggle"
        l.ZIndex         = r.ZIndex + 1
        l.Parent         = r

        local sw = Instance.new("TextButton")
        sw.AutoButtonColor = false
        sw.Text            = ""
        sw.Size            = UDim2.fromOffset(42, 22)
        sw.Position        = UDim2.new(1, -54, 0.5, -11)
        sw.BackgroundColor3 = cfg.Default and C.Accent or Color3.fromRGB(48, 50, 62)
        sw.ZIndex          = r.ZIndex + 1
        sw.Parent          = r
        corner(sw, 11)

        local knob = Instance.new("Frame")
        knob.BackgroundColor3 = Color3.fromRGB(230, 232, 240)
        knob.Size             = UDim2.fromOffset(16, 16)
        knob.Position         = cfg.Default and UDim2.new(1,-19,0.5,-8) or UDim2.fromOffset(3, 3)
        knob.ZIndex           = sw.ZIndex + 1
        knob.Parent           = sw
        corner(knob, 8)

        local state = cfg.Default and true or false
        onClick(sw, function()
            state = not state
            tw(sw,   {BackgroundColor3 = state and C.Accent or Color3.fromRGB(48,50,62)}, 0.14)
            tw(knob, {Position = state and UDim2.new(1,-19,0.5,-8) or UDim2.fromOffset(3,3)}, 0.18, Enum.EasingStyle.Back)
            if cfg.Callback then cfg.Callback(state) end
        end)
        return { Get = function() return state end }
    end

    function Api:AddSlider(cfg)
        cfg = cfg or {}
        local minv, maxv = cfg.Min or 0, cfg.Max or 100
        local value      = cfg.Default or minv
        local wrap       = row(60)

        local l = Instance.new("TextLabel")
        l.BackgroundTransparency = 1
        l.Position       = UDim2.fromOffset(14, 8)
        l.Size           = UDim2.new(0.65, 0, 0, 16)
        l.Font           = Enum.Font.Gotham
        l.TextSize       = 13
        l.TextXAlignment = Enum.TextXAlignment.Left
        l.TextColor3     = C.Text
        l.Text           = cfg.Name or "Slider"
        l.ZIndex         = wrap.ZIndex + 1
        l.Parent         = wrap

        local val = Instance.new("TextLabel")
        val.BackgroundTransparency = 1
        val.Position       = UDim2.new(0.65, 0, 0, 8)
        val.Size           = UDim2.new(0.35, -14, 0, 16)
        val.Font           = Enum.Font.GothamBold
        val.TextSize       = 13
        val.TextXAlignment = Enum.TextXAlignment.Right
        val.TextColor3     = C.Chrome
        val.Text           = tostring(value)
        val.ZIndex         = wrap.ZIndex + 1
        val.Parent         = wrap

        local bar = Instance.new("TextButton")
        bar.AutoButtonColor = false
        bar.Text            = ""
        bar.BackgroundColor3 = Color3.fromRGB(30, 32, 42)
        bar.Position        = UDim2.fromOffset(14, 36)
        bar.Size            = UDim2.new(1, -28, 0, 8)
        bar.ZIndex          = wrap.ZIndex + 1
        bar.Parent          = wrap
        corner(bar, 4)

        local fill = Instance.new("Frame")
        fill.BackgroundColor3 = C.Accent
        fill.Size             = UDim2.new((value - minv) / math.max(maxv - minv, 1), 0, 1, 0)
        fill.BorderSizePixel  = 0
        fill.ZIndex           = bar.ZIndex + 1
        fill.Parent           = bar
        corner(fill, 4)

        local knobDot = Instance.new("Frame")
        knobDot.BackgroundColor3 = C.Chrome
        knobDot.Size             = UDim2.fromOffset(10, 10)
        knobDot.AnchorPoint      = Vector2.new(0.5, 0.5)
        knobDot.Position         = UDim2.new(fill.Size.X.Scale, 0, 0.5, 0)
        knobDot.BorderSizePixel  = 0
        knobDot.ZIndex           = bar.ZIndex + 2
        knobDot.Parent           = bar
        corner(knobDot, 5)

        local sliding = false
        local function apply(x)
            local a = math.clamp((x - bar.AbsolutePosition.X) / math.max(bar.AbsoluteSize.X, 1), 0, 1)
            value    = math.floor(minv + (maxv - minv) * a + 0.5)
            fill.Size        = UDim2.new(a, 0, 1, 0)
            knobDot.Position = UDim2.new(a, 0, 0.5, 0)
            val.Text         = tostring(value)
            if cfg.Callback then cfg.Callback(value) end
        end
        bar.InputBegan:Connect(function(i)
            if i.UserInputType == Enum.UserInputType.MouseButton1 then
                playClick(); sliding = true; apply(i.Position.X)
            end
        end)
        UIS.InputEnded:Connect(function(i)
            if i.UserInputType == Enum.UserInputType.MouseButton1 then sliding = false end
        end)
        UIS.InputChanged:Connect(function(i)
            if sliding and i.UserInputType == Enum.UserInputType.MouseMovement then apply(i.Position.X) end
        end)
        return { Get = function() return value end }
    end

    function Api:AddDropdown(cfg)
        cfg = cfg or {}
        local options = cfg.Options or {"A", "B"}
        local current = cfg.Default or options[1]
        local open    = false

        local wrap = Instance.new("Frame")
        wrap.BackgroundTransparency = 1
        wrap.Size             = UDim2.new(1, 0, 0, 40)
        wrap.ClipsDescendants = true
        wrap.ZIndex           = (parent.ZIndex or 8) + 1
        wrap.Parent           = parent

        local main = Instance.new("TextButton")
        main.BackgroundColor3 = C.Raised
        main.Size             = UDim2.new(1, 0, 0, 40)
        main.Font             = Enum.Font.Gotham
        main.TextSize         = 14
        main.TextColor3       = C.Text
        main.Text             = (cfg.Name or "Select") .. "  ·  " .. tostring(current)
        main.AutoButtonColor  = false
        main.ZIndex           = wrap.ZIndex + 1
        main.Parent           = wrap
        corner(main, 8)
        stroke(main, C.Line, 1, 0.45)

        local arrow = Instance.new("TextLabel")
        arrow.BackgroundTransparency = 1
        arrow.Size           = UDim2.fromOffset(20, 40)
        arrow.Position       = UDim2.new(1, -26, 0, 0)
        arrow.Font           = Enum.Font.GothamBold
        arrow.TextSize       = 12
        arrow.Text           = "▾"
        arrow.TextColor3     = C.Mute
        arrow.ZIndex         = main.ZIndex + 1
        arrow.Parent         = main

        local list = Instance.new("Frame")
        list.BackgroundColor3 = C.Panel
        list.Position         = UDim2.fromOffset(0, 44)
        list.Size             = UDim2.new(1, 0, 0, 0)
        list.ClipsDescendants = true
        list.ZIndex           = wrap.ZIndex + 2
        list.Parent           = wrap
        corner(list, 8)
        stroke(list, C.Line, 1, 0.35)
        local ll = Instance.new("UIListLayout")
        ll.Parent = list

        local function rebuild()
            for _, c in pairs(list:GetChildren()) do
                if c:IsA("TextButton") then c:Destroy() end
            end
            for _, opt in ipairs(options) do
                local ob = Instance.new("TextButton")
                ob.BackgroundColor3 = C.Panel
                ob.Size             = UDim2.new(1, 0, 0, 32)
                ob.Font             = Enum.Font.Gotham
                ob.TextSize         = 13
                ob.TextColor3       = C.Dim
                ob.Text             = tostring(opt)
                ob.AutoButtonColor  = false
                ob.ZIndex           = list.ZIndex + 1
                ob.Parent           = list
                hover(ob, C.Panel, C.Hover)
                onClick(ob, function()
                    current   = opt
                    main.Text = (cfg.Name or "Select") .. "  ·  " .. tostring(current)
                    open      = false
                    tw(arrow, {Rotation = 0}, 0.15)
                    tw(wrap,  {Size = UDim2.new(1, 0, 0, 40)}, 0.18)
                    tw(list,  {Size = UDim2.new(1, 0, 0, 0)}, 0.18)
                    if cfg.Callback then cfg.Callback(current) end
                end)
            end
        end
        rebuild()

        onClick(main, function()
            open = not open
            local h = open and (#options * 32) or 0
            tw(arrow, {Rotation = open and 180 or 0}, 0.15)
            tw(wrap,  {Size = UDim2.new(1, 0, 0, 40 + (open and h + 8 or 0))}, 0.2)
            tw(list,  {Size = UDim2.new(1, 0, 0, h)}, 0.2)
        end)

        return {
            Get        = function() return current end,
            SetOptions = function(newOpts) options = newOpts; rebuild() end,
        }
    end

    function Api:AddKeybind(cfg)
        cfg = cfg or {}
        local bind     = cfg.Default or Enum.KeyCode.E
        local listening = false
        local r        = row(40)

        local l = Instance.new("TextLabel")
        l.BackgroundTransparency = 1
        l.Position       = UDim2.fromOffset(14, 0)
        l.Size           = UDim2.new(0.55, 0, 1, 0)
        l.Font           = Enum.Font.Gotham
        l.TextSize       = 14
        l.TextXAlignment = Enum.TextXAlignment.Left
        l.TextColor3     = C.Text
        l.Text           = cfg.Name or "Keybind"
        l.ZIndex         = r.ZIndex + 1
        l.Parent         = r

        local kb = Instance.new("TextButton")
        kb.BackgroundColor3 = C.Bg
        kb.Position         = UDim2.new(1, -102, 0.5, -13)
        kb.Size             = UDim2.fromOffset(88, 26)
        kb.Font             = Enum.Font.GothamBold
        kb.TextSize         = 12
        kb.TextColor3       = C.Chrome
        kb.AutoButtonColor  = false
        kb.ZIndex           = r.ZIndex + 1
        kb.Parent           = r
        corner(kb, 6)
        stroke(kb, C.Line, 1, 0.4)

        local function updateDisplay()
            kb.Text = typeof(bind) == "EnumItem" and bind.Name or "?"
        end
        updateDisplay()

        onClick(kb, function()
            listening = true
            kb.Text   = "..."
            tw(kb, {TextColor3 = C.Accent}, 0.1)
        end)

        UIS.InputBegan:Connect(function(input, gpe)
            if State.Destroyed then return end
            if listening then
                local newBind = nil
                if input.UserInputType == Enum.UserInputType.Keyboard then
                    if input.KeyCode ~= Enum.KeyCode.Unknown then newBind = input.KeyCode end
                elseif input.UserInputType == Enum.UserInputType.MouseButton1
                    or input.UserInputType == Enum.UserInputType.MouseButton2
                    or input.UserInputType == Enum.UserInputType.MouseButton3 then
                    newBind = input.UserInputType
                end
                if newBind then
                    bind      = newBind
                    listening = false
                    tw(kb, {TextColor3 = C.Chrome}, 0.1)
                    updateDisplay()
                    playClick()
                    if cfg.Callback then cfg.Callback(bind) end
                end
            elseif not gpe then
                if bindMatch(bind, input) then
                    if cfg.Pressed then cfg.Pressed() end
                end
            end
        end)
        return {
            Get = function() return bind end,
            Set = function(v) bind = v; updateDisplay() end,
        }
    end

    function Api:AddTextbox(cfg)
        cfg = cfg or {}
        local r   = row(40)
        local box = Instance.new("TextBox")
        box.BackgroundTransparency = 1
        box.Size             = UDim2.new(1, -24, 1, 0)
        box.Position         = UDim2.fromOffset(14, 0)
        box.Font             = Enum.Font.Gotham
        box.TextSize         = 14
        box.TextXAlignment   = Enum.TextXAlignment.Left
        box.TextColor3       = C.Text
        box.PlaceholderColor3 = C.Mute
        box.PlaceholderText  = cfg.Placeholder or cfg.Name or "Text"
        box.Text             = cfg.Default or ""
        box.ClearTextOnFocus = false
        box.ZIndex           = r.ZIndex + 1
        box.Parent           = r
        box.FocusLost:Connect(function()
            if cfg.Callback then cfg.Callback(box.Text) end
        end)
        return box
    end

    return Api
end

-- =====================================================================
--  KEY GATE / LOADING
-- =====================================================================
local function showKeyGate(done)
    local dim = Instance.new("Frame")
    dim.BackgroundColor3    = Color3.new(0, 0, 0)
    dim.BackgroundTransparency = 0.35
    dim.Size                = UDim2.fromScale(1, 1)
    dim.ZIndex              = 40
    dim.Parent              = Gui

    local card = Instance.new("Frame")
    card.AnchorPoint        = Vector2.new(0.5, 0.5)
    card.Position           = UDim2.fromScale(0.5, 0.5)
    card.Size               = UDim2.fromOffset(400, 340)
    card.BackgroundColor3   = C.Bg
    card.ZIndex             = 41
    card.Parent             = dim
    corner(card, 12)
    stroke(card, C.Steel, 1, 0.25)
    gradient(card, C.Bg, C.Void, 120)

    local icon = makeLogo(card, 84, 43)
    icon.AnchorPoint = Vector2.new(0.5, 0)
    icon.Position    = UDim2.new(0.5, 0, 0, 16)

    local name = Instance.new("TextLabel")
    name.BackgroundTransparency = 1
    name.Position  = UDim2.fromOffset(0, 104)
    name.Size      = UDim2.new(1, 0, 0, 20)
    name.Font      = Enum.Font.GothamBold
    name.TextSize  = 14
    name.TextColor3 = C.Steel
    name.Text      = "VODIUM"
    name.ZIndex    = 43
    name.Parent    = card

    local hint = Instance.new("TextLabel")
    hint.BackgroundTransparency = 1
    hint.Position  = UDim2.fromOffset(28, 128)
    hint.Size      = UDim2.new(1, -56, 0, 16)
    hint.Font      = Enum.Font.Gotham
    hint.TextSize  = 12
    hint.TextColor3 = C.Mute
    hint.Text      = "Enter license key"
    hint.ZIndex    = 43
    hint.Parent    = card

    local box = Instance.new("TextBox")
    box.BackgroundColor3  = C.Raised
    box.Position          = UDim2.fromOffset(28, 156)
    box.Size              = UDim2.new(1, -56, 0, 40)
    box.Font              = Enum.Font.Code
    box.TextSize          = 14
    box.Text              = ""
    box.PlaceholderText   = "VODIUM-XXXX-XXXX-XXXX"
    box.PlaceholderColor3 = C.Mute
    box.TextColor3        = C.Chrome
    box.ClearTextOnFocus  = false
    box.ZIndex            = 43
    box.Parent            = card
    corner(box, 8)
    local bst = stroke(box, C.Line, 1, 0.3)

    local status = Instance.new("TextLabel")
    status.BackgroundTransparency = 1
    status.Position  = UDim2.fromOffset(28, 202)
    status.Size      = UDim2.new(1, -56, 0, 16)
    status.Font      = Enum.Font.Gotham
    status.TextSize  = 12
    status.TextColor3 = C.Mute
    status.Text      = ""
    status.ZIndex    = 43
    status.Parent    = card

    local unlock = Instance.new("TextButton")
    unlock.BackgroundColor3 = C.Chrome
    unlock.Position         = UDim2.fromOffset(28, 230)
    unlock.Size             = UDim2.new(1, -56, 0, 44)
    unlock.Font             = Enum.Font.GothamBold
    unlock.TextSize         = 14
    unlock.Text             = "AUTHENTICATE"
    unlock.TextColor3       = C.Void
    unlock.AutoButtonColor  = false
    unlock.ZIndex           = 43
    unlock.Parent           = card
    corner(unlock, 8)

    local function tryUnlock()
        playClick()
        local typed = string.upper((box.Text or ""):gsub("%s+", ""))
        if typed == VALID_KEY then
            status.TextColor3 = C.Good
            status.Text       = "Accepted"
            playValid()
            task.wait(0.45)
            dim:Destroy()
            done()
        else
            status.TextColor3 = C.Bad
            status.Text       = "Rejected"
            tw(bst, {Color = C.Bad, Transparency = 0}, 0.1)
            task.wait(0.5)
            pcall(function() LocalPlayer:Kick(KICK_MESSAGE) end)
        end
    end

    onClick(unlock, tryUnlock)
    box.FocusLost:Connect(function(enter) if enter then tryUnlock() end end)
end

local function showLoading(done)
    local layer = Instance.new("Frame")
    layer.BackgroundColor3    = C.Void
    layer.BackgroundTransparency = 0.15
    layer.Size                = UDim2.fromScale(1, 1)
    layer.ZIndex              = 50
    layer.Parent              = Gui

    local panel = Instance.new("Frame")
    panel.AnchorPoint       = Vector2.new(0.5, 0.5)
    panel.Position          = UDim2.fromScale(0.5, 0.5)
    panel.Size              = UDim2.fromOffset(360, 230)
    panel.BackgroundColor3  = C.Bg
    panel.ZIndex            = 51
    panel.Parent            = layer
    corner(panel, 12)
    stroke(panel, C.Steel, 1, 0.28)

    local icon = makeLogo(panel, 72, 53)
    icon.AnchorPoint = Vector2.new(0.5, 0)
    icon.Position    = UDim2.new(0.5, 0, 0, 14)

    local brand = Instance.new("TextLabel")
    brand.BackgroundTransparency = 1
    brand.Position  = UDim2.fromOffset(0, 90)
    brand.Size      = UDim2.new(1, 0, 0, 18)
    brand.Font      = Enum.Font.GothamBold
    brand.TextSize  = 13
    brand.TextColor3 = C.Steel
    brand.Text      = "VODIUM"
    brand.ZIndex    = 53
    brand.Parent    = panel

    local status = Instance.new("TextLabel")
    status.BackgroundTransparency = 1
    status.Position  = UDim2.fromOffset(24, 118)
    status.Size      = UDim2.new(1, -48, 0, 16)
    status.Font      = Enum.Font.Gotham
    status.TextSize  = 12
    status.TextColor3 = C.Dim
    status.Text      = "Loading interface"
    status.ZIndex    = 53
    status.Parent    = panel

    local pct = Instance.new("TextLabel")
    pct.BackgroundTransparency = 1
    pct.Position       = UDim2.fromOffset(24, 138)
    pct.Size           = UDim2.new(1, -48, 0, 14)
    pct.Font           = Enum.Font.Code
    pct.TextSize       = 12
    pct.TextXAlignment = Enum.TextXAlignment.Right
    pct.TextColor3     = C.Chrome
    pct.Text           = "0%"
    pct.ZIndex         = 53
    pct.Parent         = panel

    local track = Instance.new("Frame")
    track.BackgroundColor3 = C.Raised
    track.Position         = UDim2.fromOffset(24, 168)
    track.Size             = UDim2.new(1, -48, 0, 6)
    track.ZIndex           = 53
    track.Parent           = panel
    corner(track, 3)

    local fill = Instance.new("Frame")
    fill.BackgroundColor3 = C.Accent
    fill.Size             = UDim2.new(0, 0, 1, 0)
    fill.BorderSizePixel  = 0
    fill.ZIndex           = 54
    fill.Parent           = track
    corner(fill, 3)

    local steps = {
        {0.15, "Verifying Key"},
        {0.35, "Running up Vodium"},
        {0.55, "Taking your Roblox Account"},
        {0.75, "Raping Players"},
        {1.00, "Cummed everywhere"},
    }

    task.spawn(function()
        local t0 = os.clock()
        local step = 1
        while true do
            local a = math.clamp((os.clock() - t0) / LOAD_TIME, 0, 1)
            local e = 1 - (1 - a)^3
            fill.Size = UDim2.new(e, 0, 1, 0)
            pct.Text  = tostring(math.floor(e * 100 + 0.5)) .. "%"
            if steps[step] and e >= steps[step][1] then
                status.Text = steps[step][2]
                step = step + 1
            end
            if a >= 1 then break end
            RunService.RenderStepped:Wait()
        end
        layer:Destroy()
        done()
    end)
end

-- =====================================================================
--  WINDOW
-- =====================================================================
function Library:CreateWindow(opts)
    opts = opts or {}
    local title = opts.Title or "Vodium"
    local ready = false
    local WindowApi = {}

    local function build()
        local Window = Instance.new("Frame")
        Window.Name             = "Window"
        Window.BackgroundColor3 = C.Bg
        Window.Size             = UDim2.fromOffset(660, 460)
        Window.Position         = UDim2.fromScale(0.5, 0.5)
        Window.AnchorPoint      = Vector2.new(0.5, 0.5)
        Window.ClipsDescendants = true
        Window.ZIndex           = 5
        Window.Parent           = Gui
        corner(Window, 12)
        stroke(Window, C.Line, 1, 0.3)

        local Header = Instance.new("Frame")
        Header.BackgroundColor3 = C.Panel
        Header.Size             = UDim2.new(1, 0, 0, 52)
        Header.BorderSizePixel  = 0
        Header.ZIndex           = 6
        Header.Parent           = Window

        local brand = makeLogo(Header, 32, 8)
        brand.Position = UDim2.fromOffset(12, 10)

        local Title = Instance.new("TextLabel")
        Title.BackgroundTransparency = 1
        Title.Position       = UDim2.fromOffset(50, 0)
        Title.Size           = UDim2.new(1, -160, 1, 0)
        Title.Font           = Enum.Font.GothamBold
        Title.TextSize       = 16
        Title.TextXAlignment = Enum.TextXAlignment.Left
        Title.TextColor3     = C.Text
        Title.Text           = title
        Title.ZIndex         = 8
        Title.Parent         = Header

        local hint = Instance.new("TextLabel")
        hint.BackgroundTransparency = 1
        hint.Position    = UDim2.new(1, -178, 0, 0)
        hint.Size        = UDim2.fromOffset(70, 52)
        hint.Font        = Enum.Font.Gotham
        hint.TextSize    = 11
        hint.TextColor3  = C.Mute
        hint.Text        = "LeftAlt"
        hint.ZIndex      = 8
        hint.Parent      = Header

        local Hide = Instance.new("TextButton")
        Hide.BackgroundTransparency = 1
        Hide.Size       = UDim2.fromOffset(40, 52)
        Hide.Position   = UDim2.new(1, -88, 0, 0)
        Hide.Font       = Enum.Font.GothamBold
        Hide.TextSize   = 20
        Hide.Text       = "–"
        Hide.TextColor3 = C.Dim
        Hide.ZIndex     = 8
        Hide.Parent     = Header
        Hide.MouseEnter:Connect(function() tw(Hide, {TextColor3 = C.Steel}, 0.1) end)
        Hide.MouseLeave:Connect(function() tw(Hide, {TextColor3 = C.Dim},   0.1) end)

        local Close = Instance.new("TextButton")
        Close.BackgroundTransparency = 1
        Close.Size       = UDim2.fromOffset(48, 52)
        Close.Position   = UDim2.new(1, -48, 0, 0)
        Close.Font       = Enum.Font.GothamBold
        Close.TextSize   = 18
        Close.Text       = "✕"
        Close.TextColor3 = C.Dim
        Close.ZIndex     = 8
        Close.Parent     = Header
        Close.MouseEnter:Connect(function() tw(Close, {TextColor3 = C.Bad},  0.12) end)
        Close.MouseLeave:Connect(function() tw(Close, {TextColor3 = C.Dim},  0.12) end)

        makeDrag(Header, Window)

        local Side = Instance.new("Frame")
        Side.BackgroundColor3 = C.Side
        Side.Position         = UDim2.fromOffset(0, 52)
        Side.Size             = UDim2.new(0, 158, 1, -52)
        Side.BorderSizePixel  = 0
        Side.ZIndex           = 6
        Side.Parent           = Window

        local sList = Instance.new("UIListLayout")
        sList.Padding     = UDim.new(0, 6)
        sList.SortOrder   = Enum.SortOrder.LayoutOrder
        sList.Parent      = Side
        pad(Side, 12)

        local Content = Instance.new("ScrollingFrame")
        Content.BackgroundColor3     = C.Bg
        Content.BorderSizePixel      = 0
        Content.Position             = UDim2.fromOffset(158, 52)
        Content.Size                 = UDim2.new(1, -158, 1, -52)
        Content.CanvasSize           = UDim2.new(0, 0, 0, 0)
        Content.AutomaticCanvasSize  = Enum.AutomaticSize.Y
        Content.ScrollBarThickness   = 3
        Content.ScrollBarImageColor3 = C.Accent
        Content.ZIndex               = 6
        Content.Parent               = Window

        local pageHolder = Instance.new("Frame")
        pageHolder.BackgroundTransparency = 1
        pageHolder.Size          = UDim2.new(1, 0, 0, 0)
        pageHolder.AutomaticSize = Enum.AutomaticSize.Y
        pageHolder.ZIndex        = 7
        pageHolder.Parent        = Content

        local Mini = Instance.new("ImageButton")
        Mini.Size             = UDim2.fromOffset(52, 52)
        Mini.Position         = UDim2.fromOffset(18, 200)
        Mini.BackgroundColor3 = C.Panel
        Mini.Image            = LOGO_THUMB
        Mini.ScaleType        = Enum.ScaleType.Fit
        Mini.Visible          = false
        Mini.ZIndex           = 12
        Mini.Parent           = Gui
        corner(Mini, 12)
        stroke(Mini, C.Steel, 1, 0.25)
        makeDrag(Mini, Mini)
        Mini.MouseEnter:Connect(function() tw(Mini, {BackgroundColor3 = C.Hover},  0.12) end)
        Mini.MouseLeave:Connect(function() tw(Mini, {BackgroundColor3 = C.Panel},  0.12) end)

        local function setOpen(open)
            if open then
                Mini.Visible   = false
                Window.Visible = true
            else
                Window.Visible = false
                Mini.Visible   = true
            end
        end

        State.ToggleOpen = setOpen

        onClick(Close, function() setOpen(false) end)
        onClick(Hide,  function() setOpen(false) end)

        Mini.MouseButton1Click:Connect(function()
            playClick()
            setOpen(true)
        end)

        Mini.MouseButton2Click:Connect(function()
            pcall(function() WindowApi:Destroy() end)
        end)

        conn("AltToggle", UIS.InputBegan:Connect(function(input, gpe)
            if State.Destroyed then return end
            if input.UserInputType ~= Enum.UserInputType.Keyboard then return end
            if input.KeyCode ~= TOGGLE_CLOSE then return end
            local now = os.clock()
            if now - State.LastAltToggle < 0.08 then return end
            State.LastAltToggle = now
            setOpen(not Window.Visible)
        end))

        local Pages = {}
        local Active
        local order = 0

        function WindowApi:AddTab(name)
            order = order + 1
            local btn = Instance.new("TextButton")
            btn.BackgroundColor3 = C.Raised
            btn.Size             = UDim2.new(1, 0, 0, 38)
            btn.Font             = Enum.Font.GothamBold
            btn.TextSize         = 13
            btn.TextColor3       = C.Dim
            btn.Text             = name
            btn.AutoButtonColor  = false
            btn.LayoutOrder      = order
            btn.ZIndex           = 8
            btn.Parent           = Side
            corner(btn, 8)

            local indicator = Instance.new("Frame")
            indicator.BackgroundColor3    = C.Accent
            indicator.BackgroundTransparency = 1
            indicator.Size                = UDim2.fromOffset(3, 16)
            indicator.Position            = UDim2.fromOffset(6, 11)
            indicator.BorderSizePixel     = 0
            indicator.ZIndex              = 9
            indicator.Parent              = btn
            corner(indicator, 1)

            local page = Instance.new("Frame")
            page.BackgroundTransparency = 1
            page.Size          = UDim2.new(1, 0, 0, 0)
            page.AutomaticSize = Enum.AutomaticSize.Y
            page.Visible       = false
            page.ZIndex        = 8
            page.Parent        = pageHolder
            local lay = Instance.new("UIListLayout")
            lay.Padding   = UDim.new(0, 12)
            lay.SortOrder = Enum.SortOrder.LayoutOrder
            lay.Parent    = page
            pad(page, 16)

            Pages[name] = {Btn = btn, Page = page, Ind = indicator}

            local function activate()
                for _, rec in pairs(Pages) do
                    rec.Page.Visible = false
                    rec.Btn.BackgroundColor3 = C.Raised
                    rec.Btn.TextColor3       = C.Dim
                    rec.Ind.BackgroundTransparency = 1
                end
                page.Visible             = true
                btn.BackgroundColor3     = Color3.fromRGB(28, 30, 42)
                btn.TextColor3           = C.Chrome
                indicator.BackgroundTransparency = 0
                Active = name
                Content.CanvasPosition   = Vector2.new(0, 0)
            end

            onClick(btn, activate)
            if not Active then activate() end

            local Tab = makeWidgets(page)

            function Tab:AddSection(text)
                local card = Instance.new("Frame")
                card.BackgroundColor3 = C.Panel
                card.Size             = UDim2.new(1, 0, 0, 0)
                card.AutomaticSize    = Enum.AutomaticSize.Y
                card.ZIndex           = 9
                card.Parent           = page
                corner(card, 10)
                stroke(card, C.Line, 1, 0.4)

                local sAccent = Instance.new("Frame")
                sAccent.BackgroundColor3 = C.Accent
                sAccent.BorderSizePixel  = 0
                sAccent.Size             = UDim2.fromOffset(3, 20)
                sAccent.Position         = UDim2.fromOffset(10, 8)
                sAccent.ZIndex           = 10
                sAccent.Parent           = card
                corner(sAccent, 1)

                local head = Instance.new("TextLabel")
                head.BackgroundTransparency = 1
                head.Size            = UDim2.new(1, -30, 0, 28)
                head.Position        = UDim2.fromOffset(22, 4)
                head.Font            = Enum.Font.GothamBold
                head.TextSize        = 11
                head.TextXAlignment  = Enum.TextXAlignment.Left
                head.TextColor3      = C.Steel
                head.Text            = string.upper(text or "SECTION")
                head.ZIndex          = 10
                head.Parent          = card

                local body = Instance.new("Frame")
                body.BackgroundTransparency = 1
                body.Position        = UDim2.fromOffset(0, 30)
                body.Size            = UDim2.new(1, 0, 0, 0)
                body.AutomaticSize   = Enum.AutomaticSize.Y
                body.ZIndex          = 10
                body.Parent          = card
                local bl = Instance.new("UIListLayout")
                bl.Padding   = UDim.new(0, 8)
                bl.SortOrder = Enum.SortOrder.LayoutOrder
                bl.Parent    = body
                pad(body, 0, 12, 12, 12)

                return makeWidgets(body)
            end

            return Tab
        end

        function WindowApi:Destroy()
            if State.Destroyed then return end
            State.Destroyed = true

            for _, c in pairs(State.Connections) do
                pcall(function() c:Disconnect() end)
            end
            State.Connections = {}

            pcall(function() RunService:UnbindFromRenderStep(AIMBOT_BIND_NAME) end)
            pcall(function() RunService:UnbindFromRenderStep(TP_BIND_NAME)     end)
            pcall(function() restoreTP() end)
            pcall(function() aaCleanup() end)
            pcall(function() UIS.MouseBehavior = Enum.MouseBehavior.Default end)
            pcall(function() clearEsp() end)
            pcall(function()
                if Camera.CameraType == Enum.CameraType.Scriptable then
                    Camera.CameraType = Enum.CameraType.Custom
                end
            end)
            pcall(function()
                local char = LocalPlayer.Character
                if char then
                    for _, d in pairs(char:GetDescendants()) do
                        if d:IsA("BasePart") then d.LocalTransparencyModifier = 0 end
                    end
                    local hum = char:FindFirstChildOfClass("Humanoid")
                    if hum then hum.AutoRotate = true end
                end
            end)
            pcall(function()
                LocalPlayer.CameraMinZoomDistance = 0.5
                LocalPlayer.CameraMaxZoomDistance = 400
            end)
            if State.FovDrawing then
                pcall(function() State.FovDrawing:Remove() end)
                State.FovDrawing = nil
            end
            if State.VibrantCC then
                pcall(function() State.VibrantCC:Destroy() end)
                State.VibrantCC = nil
            end
            pcall(function()
                local atm = getAtmosphere()
                if atm and State.OriginalAtmosphere then
                    atm.Density = State.OriginalAtmosphere.Density
                    atm.Haze    = State.OriginalAtmosphere.Haze
                end
            end)
            pcall(function() Gui:Destroy() end)
        end

        Library:Notify("Vodium", "Loaded. All systems green.", 3)
        ready = true
    end

    if opts.Key == false then
        showLoading(build)
    else
        showKeyGate(function() showLoading(build) end)
    end

    while not ready do task.wait() end
    return WindowApi
end

-- =====================================================================
--  BUILD UI
-- =====================================================================
task.spawn(function()
    local Window = Library:CreateWindow({ Title = "Vodium" })

    -- ── ESP ─────────────────────────────────────────────────────────
    local EspTab  = Window:AddTab("ESP")
    local EspMain = EspTab:AddSection("ESP")
    EspMain:AddToggle({ Name = "ESP Enable",  Default = false, Callback = function(v) F.Esp       = v end })
    EspMain:AddToggle({ Name = "Teamcheck",   Default = false, Callback = function(v) F.TeamCheck  = v end })
    EspMain:AddToggle({ Name = "Boxes",       Default = false, Callback = function(v) F.Boxes      = v end })
    EspMain:AddToggle({ Name = "Fill",        Default = false, Callback = function(v) F.Fill       = v end })
    EspMain:AddToggle({ Name = "Distance",    Default = false, Callback = function(v) F.Distance   = v end })
    EspMain:AddToggle({ Name = "Name",        Default = false, Callback = function(v) F.NameEsp    = v end })
    EspMain:AddToggle({ Name = "HP",          Default = false, Callback = function(v) F.HpEsp      = v end })
    EspMain:AddToggle({ Name = "Skeletons",   Default = false, Callback = function(v) F.Skeletons  = v end })
    EspMain:AddSlider({ Name = "Max Distance (studs, 0=unlimited)", Min = 0, Max = 5000, Default = 1000,
        Callback = function(n) F.EspDistance = n end })

    -- ── COMBAT ──────────────────────────────────────────────────────
    local CombatTab = Window:AddTab("Combat")
    local Aim       = CombatTab:AddSection("Aimbot")
    Aim:AddToggle({ Name = "Aimbot", Default = false,
        Callback = function(v) F.Aimbot = v; if not v then
            State.AimToggled = false
            State.LockedTarget = nil
        end end })
    Aim:AddToggle({ Name = "FOV Circle", Default = false, Callback = function(v) F.FovCircle = v end })
    Aim:AddSlider({ Name = "FOV (pixels)", Min = 10, Max = 500, Default = 120, Callback = function(n) F.Fov = n end })
    Aim:AddDropdown({ Name = "Hitbox", Options = {"Head","HumanoidRootPart","Torso","UpperTorso"},
        Default = "Head", Callback = function(v) F.AimPart = v end })
    Aim:AddDropdown({ Name = "Priority",
        Options = {"Crosshair","Closest","LowestHP","HighestHP","Facing"},
        Default = "Crosshair", Callback = function(v) F.PriorityMode = v end })
    Aim:AddDropdown({ Name = "Activation Mode", Options = {"Always","Hold","Toggle"},
        Default = "Hold", Callback = function(v) F.AimMode = v; State.AimToggled = false end })
    Aim:AddKeybind({ Name = "Aim Key (Mouse supported)", Default = Enum.UserInputType.MouseButton2,
        Callback = function(k) F.AimKey = k end })
    Aim:AddToggle({ Name = "Team Check",  Default = false, Callback = function(v) F.AimbotTeamCheck = v end })
    Aim:AddToggle({ Name = "Wall Check",  Default = false, Callback = function(v) F.WallCheck        = v end })

    local AimAdv = CombatTab:AddSection("Aimbot · Smoothing & Prediction")
    AimAdv:AddDropdown({ Name = "Aim Style", Options = {"Smooth","Snap"},
        Default = "Smooth", Callback = function(v) F.AimStyle = v end })
    AimAdv:AddSlider({ Name = "Horizontal Smoothness (1=slow, 100=snap)",
        Min = 1, Max = 100, Default = 35, Callback = function(n) F.AimHorizontal = n end })
    AimAdv:AddSlider({ Name = "Vertical Smoothness (1=slow, 100=snap)",
        Min = 1, Max = 100, Default = 35, Callback = function(n) F.AimVertical = n end })
    AimAdv:AddSlider({ Name = "Humanize (reduces assist while you move mouse)",
        Min = 0, Max = 100, Default = 0, Callback = function(n) F.AimHumanize = n end })
    AimAdv:AddSlider({ Name = "Prediction (x0.01 s)", Min = 0, Max = 30, Default = 0,
        Callback = function(n) F.AimPrediction = n end })
    AimAdv:AddSlider({ Name = "Gravity Compensation (studs up)",
        Min = 0, Max = 20, Default = 0, Callback = function(n) F.AimGravityComp = n end })
    AimAdv:AddLabel("Sensitivity auto-calibrates in ~5 frames; no manual tuning needed.")

    local AimTgt = CombatTab:AddSection("Aimbot · Target Lock")
    AimTgt:AddToggle({ Name = "Target Stickiness (don't swap mid-fight)",
        Default = true, Callback = function(v) F.AimStick = v; if not v then State.LockedTarget = nil end end })
    AimTgt:AddSlider({ Name = "Sticky FOV Tolerance (%)", Min = 100, Max = 200, Default = 130,
        Callback = function(n) F.AimStickBoost = n end })

    local Wep = CombatTab:AddSection("Weapon")
    Wep:AddToggle({ Name = "Hitbox Expander (no collision)", Default = false,
        Callback = function(v) F.HitboxExpander = v end })
    Wep:AddSlider({ Name = "Expander Size", Min = 2, Max = 30, Default = 12,
        Callback = function(n) F.HitboxSize = n end })

    local Fire = CombatTab:AddSection("Auto-Fire / Trigger Bot")
    Fire:AddToggle({ Name = "Auto Fire (fires while aiming)", Default = false,
        Callback = function(v) F.AutoShoot = v end })
    Fire:AddToggle({ Name = "Auto Fire · only when target visible",
        Default = true, Callback = function(v) F.AutoShootVisible = v end })
    Fire:AddSlider({ Name = "Fire Rate (shots/sec)", Min = 1, Max = 30, Default = 10,
        Callback = function(n) F.AutoFireRate = n end })
    Fire:AddToggle({ Name = "Trigger Bot (fires only when crosshair is on target)",
        Default = false, Callback = function(v) F.TriggerBot = v end })
    Fire:AddSlider({ Name = "Trigger FOV (pixels)", Min = 1, Max = 30, Default = 3,
        Callback = function(n) F.TriggerFov = n end })
    Fire:AddToggle({ Name = "Trigger · only when target visible",
        Default = true, Callback = function(v) F.TriggerVisible = v end })

    -- ── RAGE ────────────────────────────────────────────────────────
    local RageTab = Window:AddTab("Rage")
    local AA      = RageTab:AddSection("Anti Aim")
    AA:AddToggle({ Name = "Anti Aim", Default = false,
        Callback = function(v) F.AntiAim = v; if not v then aaCleanup() end end })
    AA:AddDropdown({ Name = "Preset",
        Options = {"Custom","Legit","Semi-Rage","Rage","HvH"}, Default = "Custom",
        Callback = function(v)
            if     v == "Legit"     then F.AAYawBase="Local";   F.AAYawOffset=25;  F.AAPitchMode="None";  F.AARoll=false; F.AASpinSpeed=8
            elseif v == "Semi-Rage" then F.AAYawBase="Reverse"; F.AAYawOffset=45;  F.AAPitchMode="Down";  F.AAPitchAngle=45; F.AARoll=false
            elseif v == "Rage"      then F.AAYawBase="Spin";    F.AAYawOffset=0;   F.AAPitchMode="Down";  F.AAPitchAngle=89; F.AASpinSpeed=14; F.AARoll=false
            elseif v == "HvH"       then F.AAYawBase="Jitter";  F.AAJitterRange=90;F.AAPitchMode="Jitter";F.AAPitchAngle=60; F.AARoll=true; F.AARollSpeed=8
            end
        end })
    AA:AddDropdown({ Name = "Yaw Base",
        Options = {"Local","Reverse","Manual","Spin","Jitter","Random","Sideways"},
        Default = "Local", Callback = function(v) F.AAYawBase = v end })
    AA:AddSlider({ Name = "Yaw Offset (deg)",   Min = -180, Max = 180, Default = 0,  Callback = function(n) F.AAYawOffset  = n end })
    AA:AddSlider({ Name = "Manual Yaw (deg)",   Min = -180, Max = 180, Default = 0,  Callback = function(n) F.AAManualYaw  = n end })
    AA:AddSlider({ Name = "Jitter Range (deg)", Min = 10,   Max = 180, Default = 90, Callback = function(n) F.AAJitterRange = n end })
    AA:AddSlider({ Name = "Spin Speed",         Min = 1,    Max = 60,  Default = 14, Callback = function(n) F.AASpinSpeed  = n end })
    AA:AddDropdown({ Name = "Pitch Mode",
        Options = {"None","Down","Up","Jitter","Random"}, Default = "None",
        Callback = function(v) F.AAPitchMode = v end })
    AA:AddSlider({ Name = "Pitch Angle (deg)", Min = 0, Max = 89, Default = 89,
        Callback = function(n) F.AAPitchAngle = n end })

    local AA2 = RageTab:AddSection("Anti-Aim Extras")
    AA2:AddToggle({ Name = "Roll AA (spin on Z-axis)", Default = false,  Callback = function(v) F.AARoll      = v end })
    AA2:AddSlider({ Name = "Roll Speed",               Min = 1, Max = 20, Default = 5, Callback = function(n) F.AARollSpeed = n end })
    AA2:AddToggle({ Name = "Only AA While Moving",     Default = false,  Callback = function(v) F.AAOnMove    = v end })
    AA2:AddToggle({ Name = "Only AA While Jumping",    Default = false,  Callback = function(v) F.AAOnJump    = v end })
    AA2:AddToggle({ Name = "Force Head Rotation (First Person)", Default = false,
        Callback = function(v) F.AAFirstPerson = v end })

    local TP = RageTab:AddSection("Third Person (CS:GO style)")
    TP:AddToggle({ Name = "Third Person", Default = false,
        Callback = function(v) F.ThirdPerson = v; if not v then restoreTP() end end })
    TP:AddSlider({ Name = "Camera Distance",    Min = 3,  Max = 20,  Default = 8,  Callback = function(n) F.TPOffset   = n end })
    TP:AddSlider({ Name = "Mouse Sensitivity",  Min = 10, Max = 100, Default = 40, Callback = function(n) F.TPSens     = n end })
    TP:AddToggle({ Name = "Character Faces Camera", Default = true, Callback = function(v) F.TPFaceCamera = v end })
    TP:AddLabel("CS:GO style — mouse controls your view like FPS. You see your own body.")

    -- ── VISUALS ─────────────────────────────────────────────────────
    local VisTab = Window:AddTab("Visuals")
    local World  = VisTab:AddSection("World")
    World:AddToggle({ Name = "Fullbright", Default = false, Callback = function(v) F.Fullbright = v end })
    World:AddToggle({ Name = "NoFog",      Default = false, Callback = function(v) F.NoFog      = v end })
    World:AddToggle({ Name = "Vibrant Color", Default = false, Callback = function(v) F.Vibrant = v end })
    World:AddLabel("Fullbright also disables Atmosphere density/haze.")

    -- ── MOVEMENT ────────────────────────────────────────────────────
    local MovTab = Window:AddTab("Movement")
    local Mov    = MovTab:AddSection("Movement")
    Mov:AddToggle({ Name = "Fly",          Default = false, Callback = function(v) F.Fly       = v end })
    Mov:AddSlider({ Name = "Fly Speed",    Min = 10,  Max = 300, Default = 60, Callback = function(n) F.FlySpeed  = n end })
    Mov:AddToggle({ Name = "Speed",        Default = false, Callback = function(v) F.Speed     = v end })
    Mov:AddSlider({ Name = "WalkSpeed",    Min = 16,  Max = 500, Default = 16, Callback = function(n) F.WalkSpeed = n end })
    Mov:AddToggle({ Name = "JumpPower",    Default = false, Callback = function(v) F.JumpPower = v end })
    Mov:AddSlider({ Name = "JumpPower Value", Min = 50, Max = 500, Default = 50, Callback = function(n) F.JPValue  = n end })
    Mov:AddToggle({ Name = "Infinity Jump", Default = false, Callback = function(v) F.InfJump  = v end })
    Mov:AddToggle({ Name = "NoClip",        Default = false, Callback = function(v) F.Noclip   = v end })
    Mov:AddToggle({ Name = "Freecam",       Default = false, Callback = function(v) F.Freecam  = v end })
    Mov:AddSlider({ Name = "Freecam Speed", Min = 1, Max = 10, Default = 1, Callback = function(n) F.FC_Speed = n end })

    -- ── TELEPORT ────────────────────────────────────────────────────
    local TpTab  = Window:AddTab("Teleport")
    local TpSec  = TpTab:AddSection("Teleport to Player")
    local playerList = {}
    local function refreshPlayers()
        table.clear(playerList)
        for _, p in pairs(Players:GetPlayers()) do
            if p ~= LocalPlayer then table.insert(playerList, p.Name) end
        end
        table.sort(playerList)
    end
    refreshPlayers()
    local tpDrop = TpSec:AddDropdown({
        Name     = "Target",
        Options  = playerList,
        Default  = playerList[1] or "",
        Callback = function(v) F.TpTarget = v end,
    })
    TpSec:AddButton({ Name = "Refresh List", Callback = function()
        refreshPlayers()
        tpDrop.SetOptions(playerList)
        Library:Notify("Teleport", "Player list refreshed.", 1.5)
    end })
    TpSec:AddButton({ Name = "Teleport", Callback = function()
        local target = Players:FindFirstChild(F.TpTarget)
        if not target then Library:Notify("Teleport", "Target not found.", 2); return end
        local troot = getRoot(target)
        local lroot = getRoot(LocalPlayer)
        if troot and lroot then
            lroot.CFrame = troot.CFrame + Vector3.new(0, 3, 0)
            Library:Notify("Teleport", "Teleported to " .. target.Name, 2)
        else
            Library:Notify("Teleport", "Character not found.", 2)
        end
    end })

    -- ── MISC ────────────────────────────────────────────────────────
    local MiscTab = Window:AddTab("Misc")
    local Msc     = MiscTab:AddSection("Extra")
    Msc:AddButton({ Name = "Copy PlaceId", Callback = function()
        if setclipboard then
            setclipboard(tostring(game.PlaceId))
            Library:Notify("Clipboard", "Copied.", 2)
        else
            Library:Notify("Clipboard", "setclipboard missing.", 2)
        end
    end })
    Msc:AddKeybind({ Name = "Panic Key (unload)", Default = Enum.KeyCode.End,
        Pressed = function() Window:Destroy() end })

    local Sys = MiscTab:AddSection("System")
    Sys:AddButton({ Name = "Unload Library", Callback = function() Window:Destroy() end })
end)

return Library
