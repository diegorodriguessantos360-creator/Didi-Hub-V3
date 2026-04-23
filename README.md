--[[
    ╔══════════════════════════════════════════════════════════════╗
    ║                        DIDI HUB V3                          ║
    ║                  Developed by  46dgin                       ║
    ║              Discord: discord.gg/didihub                    ║
    ╚══════════════════════════════════════════════════════════════╝

    :: Modules  ::  ESP · Combat · Player · Misc
    :: Version  ::  3.0
    :: UI       ::  Rayfield Library
]]

-- ──────────────────────────────────────────────
--  CORE LIBRARY
-- ──────────────────────────────────────────────

local Rayfield = loadstring(game:HttpGet("https://sirius.menu/rayfield"))()

-- ──────────────────────────────────────────────
--  WINDOW  /  KEY SYSTEM
-- ──────────────────────────────────────────────

local Window = Rayfield:CreateWindow({
    Name             = "Didi Hub V3",
    Icon             = "shield",
    LoadingTitle     = "Didi Hub V3",
    LoadingSubtitle  = "Version 3.0  |  Developed by 46dgin",

    ConfigurationSaving = {
        Enabled    = true,
        FolderName = "DidiHub",
        FileName   = "DidiHubV3",
    },

    KeySystem = true,
    KeySettings = {
        Title          = "Didi Hub V3",
        Subtitle       = "Restricted Access",
        Note           = "Discord: discord.gg/didihub",
        FileName       = "Didi Hub V3 Key",
        SaveKey        = false,
        GrabKeyFromSite = false,
        Key            = { "rootmagisk" },
    },
})

-- ──────────────────────────────────────────────
--  SERVICES
-- ──────────────────────────────────────────────

local Players     = game:GetService("Players")
local RunService  = game:GetService("RunService")
local UIS         = game:GetService("UserInputService")
local Lighting    = game:GetService("Lighting")

local HttpService = game:GetService("HttpService")
local Camera      = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

local function GetChar()
    return LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
end

-- ──────────────────────────────────────────────
--  COLOR SAVE  —  SYSTEM
-- ──────────────────────────────────────────────

local COLOR_FILE = "DidiHub/DidiHubV3_Colors.json"

local function ColorToTable(c)
    return { R = math.floor(c.R * 255), G = math.floor(c.G * 255), B = math.floor(c.B * 255) }
end

local function TableToColor(t)
    return Color3.fromRGB(t.R, t.G, t.B)
end

local function SaveColors()
    local data = {}
    for key, color in pairs(ESPColors) do
        if key ~= "Rainbow" then
            data[key] = ColorToTable(color)
        end
    end
    local ok, encoded = pcall(HttpService.JSONEncode, HttpService, data)
    if ok then
        pcall(writefile, COLOR_FILE, encoded)
    end
end

local function LoadColors()
    local ok, raw = pcall(readfile, COLOR_FILE)
    if not ok or not raw then return end
    local ok2, data = pcall(HttpService.JSONDecode, HttpService, raw)
    if not ok2 or not data then return end
    for key, t in pairs(data) do
        if ESPColors[key] then
            ESPColors[key] = TableToColor(t)
        end
    end
end

-- ──────────────────────────────────────────────
--  TABS
-- ──────────────────────────────────────────────

local CombatTab = Window:CreateTab("⚔️ Combat", 4483362458)
local PlayerTab = Window:CreateTab("🏃 Player", 4483362458)
local VisualTab = Window:CreateTab("👁️ Visual", 4483362458)
local MiscTab   = Window:CreateTab("⚙️ Misc",   4483362458)

-- ──────────────────────────────────────────────
--  STATE  —  Combat / Player
-- ──────────────────────────────────────────────

local TargetLock   = false
local SmoothAmount = 0.15
local TeamCheck    = true

local FOV      = 150
local ShowFOV  = true
local FOVColor = Color3.fromRGB(255, 0, 0)

local SpeedEnabled = false
local SpeedValue   = 50
local JumpEnabled  = false
local JumpValue    = 120
local InfJump      = false
local NoClip       = false
local BunnyHop     = false

local AntiLagEnabled = false

-- ──────────────────────────────────────────────
--  ESP  —  COLOR PALETTE
-- ──────────────────────────────────────────────

local ESPColors = {
    -- Team base
    Enemy         = Color3.fromRGB(255,  25,  25),
    Ally          = Color3.fromRGB( 25, 255,  25),

    -- Per-element
    BoxEnemy      = Color3.fromRGB(255,  25,  25),
    BoxAlly       = Color3.fromRGB( 25, 255,  25),
    TracerEnemy   = Color3.fromRGB(255,  25,  25),
    TracerAlly    = Color3.fromRGB( 25, 255,  25),
    HealthBar     = Color3.fromRGB(  0, 255,   0),
    HealthText    = Color3.fromRGB(255, 255, 255),
    NameEnemy     = Color3.fromRGB(255, 255, 255),
    NameAlly      = Color3.fromRGB(255, 255, 255),
    Distance      = Color3.fromRGB(200, 200, 200),
    SnaplineEnemy = Color3.fromRGB(255,  25,  25),
    SnaplineAlly  = Color3.fromRGB( 25, 255,  25),
    Skeleton      = Color3.fromRGB(255, 255, 255),
    ChamsFill     = Color3.fromRGB(255,   0,   0),
    ChamsOccluded = Color3.fromRGB(150,   0,   0),
    ChamsOutline  = Color3.fromRGB(255, 255, 255),
    Rainbow       = Color3.fromRGB(255,   0,   0),
}

-- ──────────────────────────────────────────────
--  ESP  —  SETTINGS
-- ──────────────────────────────────────────────

local ESPSettings = {
    -- General
    Enabled             = false,
    TeamCheck           = true,
    ShowTeam            = false,
    MaxDistance         = 1000,
    TextSize            = 14,
    TextFont            = 2,

    -- Box
    BoxESP              = false,
    BoxStyle            = "Corner",   -- "Corner" | "Full"
    BoxThickness        = 1,
    BoxFillTransparency = 0.5,

    -- Tracer
    TracerESP           = false,
    TracerOrigin        = "Bottom",   -- "Bottom" | "Top" | "Mouse" | "Center"
    TracerThickness     = 1,

    -- Health
    HealthESP           = false,
    HealthStyle         = "Bar",      -- "Bar" | "Text" | "Both"
    HealthTextSuffix    = "HP",

    -- Name / Distance / Snaplines
    NameESP             = false,
    ShowDistance        = true,
    Snaplines           = false,

    -- Rainbow
    RainbowEnabled      = false,
    RainbowSpeed        = 1,

    -- Chams
    ChamsEnabled            = false,
    ChamsFillColor          = Color3.fromRGB(255,   0,   0),
    ChamsOccludedColor      = Color3.fromRGB(150,   0,   0),
    ChamsOutlineColor       = Color3.fromRGB(255, 255, 255),
    ChamsTransparency       = 0.5,
    ChamsOutlineTransparency = 0,
    ChamsOutlineThickness   = 0.1,

    -- Skeleton
    SkeletonESP         = false,
    SkeletonColor       = Color3.fromRGB(255, 255, 255),
    SkeletonThickness   = 1.5,
    SkeletonTransparency = 1,
}

local ESPDrawings  = {}
local ESPSkeleton  = {}
local ESPHighlight = {}

-- ──────────────────────────────────────────────
--  ANTILAG
-- ──────────────────────────────────────────────

local function ApplyAntiLag()
    if not AntiLagEnabled then return end

    for _, fx in pairs(Lighting:GetChildren()) do
        if fx:IsA("BloomEffect") or fx:IsA("BlurEffect")
        or fx:IsA("ColorCorrectionEffect") or fx:IsA("SunRaysEffect")
        or fx:IsA("DepthOfFieldEffect") then
            fx.Enabled = false
        end
    end

    local terrain = workspace:FindFirstChildOfClass("Terrain")
    if terrain then terrain.Decoration = false end

    for _, obj in pairs(workspace:GetDescendants()) do
        if obj:IsA("ParticleEmitter") or obj:IsA("Trail")
        or obj:IsA("Smoke") or obj:IsA("Fire") or obj:IsA("Sparkles") then
            obj.Enabled = false
        elseif obj:IsA("Explosion") then
            obj:Destroy()
        elseif obj:IsA("BasePart") then
            obj.Material    = Enum.Material.SmoothPlastic
            obj.Reflectance = 0
            obj.CastShadow  = false
        elseif obj:IsA("Decal") or obj:IsA("Texture") then
            obj.Transparency = 1
        end
    end

    settings().Rendering.QualityLevel = Enum.QualityLevel.Level01
    Rayfield:Notify({ Title = "AntiLag", Content = "Optimizations applied!", Duration = 3 })
end

local function DisableAntiLag()
    for _, fx in pairs(Lighting:GetChildren()) do
        if fx:IsA("BloomEffect") or fx:IsA("BlurEffect")
        or fx:IsA("ColorCorrectionEffect") or fx:IsA("SunRaysEffect")
        or fx:IsA("DepthOfFieldEffect") then
            fx.Enabled = true
        end
    end

    local terrain = workspace:FindFirstChildOfClass("Terrain")
    if terrain then terrain.Decoration = true end

    for _, obj in pairs(workspace:GetDescendants()) do
        if obj:IsA("ParticleEmitter") or obj:IsA("Trail")
        or obj:IsA("Smoke") or obj:IsA("Fire") or obj:IsA("Sparkles") then
            obj.Enabled = true
        elseif obj:IsA("Decal") or obj:IsA("Texture") then
            obj.Transparency = 0
        end
    end

    settings().Rendering.QualityLevel = Enum.QualityLevel.Automatic
    Rayfield:Notify({ Title = "AntiLag", Content = "Effects restored!", Duration = 3 })
end

-- ──────────────────────────────────────────────
--  FOV CIRCLE
-- ──────────────────────────────────────────────

local Circle         = Drawing.new("Circle")
Circle.Thickness     = 2
Circle.NumSides      = 100
Circle.Filled        = false
Circle.Radius        = FOV
Circle.Color         = FOVColor
Circle.Visible       = ShowFOV

-- ──────────────────────────────────────────────
--  TARGET LOCK
-- ──────────────────────────────────────────────

local function GetClosest()
    local closest, shortest = nil, FOV
    local center = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)

    for _, p in pairs(Players:GetPlayers()) do
        if p == LocalPlayer then continue end
        if not (p.Character and p.Character:FindFirstChild("Head")) then continue end
        if TeamCheck and p.Team == LocalPlayer.Team then continue end

        local pos, vis = Camera:WorldToViewportPoint(p.Character.Head.Position)
        if vis then
            local dist = (Vector2.new(pos.X, pos.Y) - center).Magnitude
            if dist < shortest then
                shortest = dist
                closest  = p
            end
        end
    end

    return closest
end

-- ──────────────────────────────────────────────
--  NOCLIP
-- ──────────────────────────────────────────────

RunService.Stepped:Connect(function()
    if not NoClip then return end
    local char = LocalPlayer.Character
    if not char then return end
    for _, v in pairs(char:GetDescendants()) do
        if v:IsA("BasePart") then v.CanCollide = false end
    end
end)

-- ──────────────────────────────────────────────
--  ESP  —  HELPER FUNCTIONS
-- ──────────────────────────────────────────────

local function MakeLine()
    local l   = Drawing.new("Line")
    l.Visible = false
    return l
end

local function GetESPColor(player, element)
    local isEnemy = not (ESPSettings.TeamCheck and player.Team == LocalPlayer.Team)
    if ESPSettings.RainbowEnabled then return ESPColors.Rainbow end

    local map = {
        Box      = isEnemy and ESPColors.BoxEnemy      or ESPColors.BoxAlly,
        Tracer   = isEnemy and ESPColors.TracerEnemy   or ESPColors.TracerAlly,
        HealthBar  = ESPColors.HealthBar,
        HealthText = ESPColors.HealthText,
        Name     = isEnemy and ESPColors.NameEnemy     or ESPColors.NameAlly,
        Distance   = ESPColors.Distance,
        Snapline = isEnemy and ESPColors.SnaplineEnemy or ESPColors.SnaplineAlly,
        Skeleton   = ESPColors.Skeleton,
    }

    return map[element] or (isEnemy and ESPColors.Enemy or ESPColors.Ally)
end

local function CreateESPEntry(player)
    if player == LocalPlayer then return end

    -- Box lines (8 segments)
    local box = {}
    for _, k in ipairs({ "TL","TR","BL","BR","L","R","T","B" }) do
        box[k]           = MakeLine()
        box[k].Color     = GetESPColor(player, "Box")
        box[k].Thickness = ESPSettings.BoxThickness
    end

    -- Tracer
    local tracer           = MakeLine()
    tracer.Color           = GetESPColor(player, "Tracer")
    tracer.Thickness       = ESPSettings.TracerThickness

    -- Health bar
    local hpOutline        = Drawing.new("Square")
    hpOutline.Filled       = false
    hpOutline.Visible      = false
    hpOutline.Color        = Color3.new(0, 0, 0)
    hpOutline.Thickness    = 1

    local hpFill           = Drawing.new("Square")
    hpFill.Filled          = true
    hpFill.Visible         = false
    hpFill.Color           = GetESPColor(player, "HealthBar")

    local hpText           = Drawing.new("Text")
    hpText.Center          = true
    hpText.Size            = ESPSettings.TextSize
    hpText.Color           = GetESPColor(player, "HealthText")
    hpText.Outline         = true
    hpText.Visible         = false

    -- Name & distance labels
    local nameText         = Drawing.new("Text")
    nameText.Center        = true
    nameText.Size          = ESPSettings.TextSize
    nameText.Color         = GetESPColor(player, "Name")
    nameText.Outline       = true
    nameText.Visible       = false

    local distText         = Drawing.new("Text")
    distText.Center        = true
    distText.Size          = ESPSettings.TextSize - 2
    distText.Color         = GetESPColor(player, "Distance")
    distText.Outline       = true
    distText.Visible       = false

    -- Snapline
    local snapline         = MakeLine()
    snapline.Color         = GetESPColor(player, "Snapline")

    -- Skeleton bones
    local skeleton = {}
    for _, k in ipairs({
        "Head","Neck","UpperSpine","LowerSpine",
        "LShoulder","LUpperArm","LLowerArm","LHand",
        "RShoulder","RUpperArm","RLowerArm","RHand",
        "LHip","LUpperLeg","LLowerLeg","LFoot",
        "RHip","RUpperLeg","RLowerLeg","RFoot",
    }) do
        skeleton[k]              = MakeLine()
        skeleton[k].Color        = GetESPColor(player, "Skeleton")
        skeleton[k].Thickness    = ESPSettings.SkeletonThickness
        skeleton[k].Transparency = ESPSettings.SkeletonTransparency
    end
    ESPSkeleton[player] = skeleton

    -- Highlight (Chams)
    local hl                     = Instance.new("Highlight")
    hl.FillColor                 = ESPColors.ChamsFill
    hl.OutlineColor              = ESPColors.ChamsOutline
    hl.FillTransparency          = ESPSettings.ChamsTransparency
    hl.OutlineTransparency       = ESPSettings.ChamsOutlineTransparency
    hl.DepthMode                 = Enum.HighlightDepthMode.AlwaysOnTop
    hl.Enabled                   = false
    ESPHighlight[player]         = hl

    ESPDrawings[player] = {
        Box       = box,
        Tracer    = tracer,
        HpOutline = hpOutline,
        HpFill    = hpFill,
        HpText    = hpText,
        Name      = nameText,
        Dist      = distText,
        Snap      = snapline,
    }
end

local function RemoveESPEntry(player)
    local d = ESPDrawings[player]
    if d then
        for _, v in pairs(d.Box) do v:Remove() end
        d.Tracer:Remove()
        d.HpOutline:Remove()
        d.HpFill:Remove()
        d.HpText:Remove()
        d.Name:Remove()
        d.Dist:Remove()
        d.Snap:Remove()
        ESPDrawings[player] = nil
    end

    local sk = ESPSkeleton[player]
    if sk then
        for _, l in pairs(sk) do l:Remove() end
        ESPSkeleton[player] = nil
    end

    local hl = ESPHighlight[player]
    if hl then hl:Destroy(); ESPHighlight[player] = nil end
end

local function HideAll(player)
    local d = ESPDrawings[player]
    if d then
        for _, v in pairs(d.Box) do v.Visible = false end
        d.Tracer.Visible    = false
        d.HpOutline.Visible = false
        d.HpFill.Visible    = false
        d.HpText.Visible    = false
        d.Name.Visible      = false
        d.Dist.Visible      = false
        d.Snap.Visible      = false
    end
    local sk = ESPSkeleton[player]
    if sk then for _, l in pairs(sk) do l.Visible = false end end
end

local function GetTracerOrigin()
    local cx = Camera.ViewportSize.X / 2
    local cy = Camera.ViewportSize.Y

    if ESPSettings.TracerOrigin == "Top"    then return Vector2.new(cx, 0)  end
    if ESPSettings.TracerOrigin == "Mouse"  then return UIS:GetMouseLocation() end
    if ESPSettings.TracerOrigin == "Center" then return Vector2.new(cx, cy / 2) end
    return Vector2.new(cx, cy)
end

local function DrawBone(from, to, line, player)
    if not from or not to then line.Visible = false; return end
    local a, aVis = Camera:WorldToViewportPoint(from.Position)
    local b, bVis = Camera:WorldToViewportPoint(to.Position)
    if not (aVis and bVis) or a.Z < 0 or b.Z < 0 then line.Visible = false; return end
    line.From         = Vector2.new(a.X, a.Y)
    line.To           = Vector2.new(b.X, b.Y)
    line.Color        = GetESPColor(player, "Skeleton")
    line.Thickness    = ESPSettings.SkeletonThickness
    line.Transparency = ESPSettings.SkeletonTransparency
    line.Visible      = true
end

local function UpdateESPEntry(player)
    if not ESPSettings.Enabled then HideAll(player); return end

    local d = ESPDrawings[player]
    if not d then return end

    local char = player.Character
    local hrp  = char and char:FindFirstChild("HumanoidRootPart")
    local hum  = char and char:FindFirstChildOfClass("Humanoid")
    if not hrp or not hum or hum.Health <= 0 then HideAll(player); return end

    if ESPSettings.TeamCheck and player.Team == LocalPlayer.Team and not ESPSettings.ShowTeam then
        HideAll(player); return
    end

    local pos, onScreen = Camera:WorldToViewportPoint(hrp.Position)
    local dist = (hrp.Position - Camera.CFrame.Position).Magnitude
    if not onScreen or dist > ESPSettings.MaxDistance then HideAll(player); return end

    local cf   = hrp.CFrame
    local size = char:GetExtentsSize()
    local topV, tVis = Camera:WorldToViewportPoint((cf * CFrame.new(0,  size.Y / 2, 0)).Position)
    local botV, bVis = Camera:WorldToViewportPoint((cf * CFrame.new(0, -size.Y / 2, 0)).Position)
    if not (tVis and bVis) then HideAll(player); return end

    local screenH = botV.Y - topV.Y
    local boxW    = screenH * 0.65
    local boxPos  = Vector2.new(topV.X - boxW / 2, topV.Y)
    local boxSize = Vector2.new(boxW, screenH)

    -- Reset box visibility
    for _, v in pairs(d.Box) do v.Visible = false end

    -- ── Box ──────────────────────────────────
    if ESPSettings.BoxESP then
        local col = GetESPColor(player, "Box")
        local th  = ESPSettings.BoxThickness
        local s   = ESPSettings.BoxStyle

        if s == "Corner" then
            local cs = boxW * 0.2
            local segments = {
                { d.Box.TL, boxPos,                          boxPos + Vector2.new(cs,      0)       },
                { d.Box.TR, boxPos + Vector2.new(boxW, 0),   boxPos + Vector2.new(boxW-cs, 0)       },
                { d.Box.BL, boxPos + Vector2.new(0, screenH),boxPos + Vector2.new(cs,      screenH) },
                { d.Box.BR, boxPos + boxSize,                boxPos + Vector2.new(boxW-cs, screenH) },
                { d.Box.L,  boxPos,                          boxPos + Vector2.new(0,       cs)      },
                { d.Box.R,  boxPos + Vector2.new(boxW, 0),   boxPos + Vector2.new(boxW,    cs)      },
                { d.Box.T,  boxPos + Vector2.new(0, screenH),boxPos + Vector2.new(0,       screenH-cs) },
                { d.Box.B,  boxPos + boxSize,                boxPos + Vector2.new(boxW,    screenH-cs) },
            }
            for _, seg in ipairs(segments) do
                seg[1].From      = seg[2]
                seg[1].To        = seg[3]
                seg[1].Color     = col
                seg[1].Thickness = th
                seg[1].Visible   = true
            end

        elseif s == "Full" then
            local edges = {
                { d.Box.L, boxPos,                         boxPos + Vector2.new(0,    screenH) },
                { d.Box.R, boxPos + Vector2.new(boxW, 0),  boxPos + boxSize                   },
                { d.Box.T, boxPos,                         boxPos + Vector2.new(boxW, 0)       },
                { d.Box.B, boxPos + Vector2.new(0,screenH),boxPos + boxSize                   },
            }
            for _, e in ipairs(edges) do
                e[1].From      = e[2]
                e[1].To        = e[3]
                e[1].Color     = col
                e[1].Thickness = th
                e[1].Visible   = true
            end
        end
    end

    -- ── Tracer ───────────────────────────────
    if ESPSettings.TracerESP then
        d.Tracer.From      = GetTracerOrigin()
        d.Tracer.To        = Vector2.new(pos.X, pos.Y)
        d.Tracer.Color     = GetESPColor(player, "Tracer")
        d.Tracer.Thickness = ESPSettings.TracerThickness
        d.Tracer.Visible   = true
    else
        d.Tracer.Visible = false
    end

    -- ── Health Bar ───────────────────────────
    if ESPSettings.HealthESP then
        local hp   = hum.Health / hum.MaxHealth
        local barH = screenH * 0.8
        local barW = 4
        local bx   = boxPos.X - barW - 2
        local by   = boxPos.Y + (screenH - barH) / 2

        d.HpOutline.Size     = Vector2.new(barW, barH)
        d.HpOutline.Position = Vector2.new(bx, by)
        d.HpOutline.Visible  = true

        d.HpFill.Size     = Vector2.new(barW - 2, barH * hp)
        d.HpFill.Position = Vector2.new(bx + 1, by + barH * (1 - hp))
        d.HpFill.Color    = Color3.fromRGB(255 - 255 * hp, 255 * hp, 0)
        d.HpFill.Visible  = true

        if ESPSettings.HealthStyle == "Text" or ESPSettings.HealthStyle == "Both" then
            d.HpText.Text     = math.floor(hum.Health) .. ESPSettings.HealthTextSuffix
            d.HpText.Position = Vector2.new(bx + barW + 2, by + barH / 2)
            d.HpText.Color    = GetESPColor(player, "HealthText")
            d.HpText.Visible  = true
        else
            d.HpText.Visible = false
        end
    else
        d.HpOutline.Visible = false
        d.HpFill.Visible    = false
        d.HpText.Visible    = false
    end

    -- ── Name ─────────────────────────────────
    if ESPSettings.NameESP then
        d.Name.Text     = player.DisplayName
        d.Name.Position = Vector2.new(boxPos.X + boxW / 2, boxPos.Y - 20)
        d.Name.Color    = GetESPColor(player, "Name")
        d.Name.Size     = ESPSettings.TextSize
        d.Name.Visible  = true
    else
        d.Name.Visible = false
    end

    -- ── Distance ─────────────────────────────
    if ESPSettings.ShowDistance then
        d.Dist.Text     = math.floor(dist) .. "m"
        d.Dist.Position = Vector2.new(boxPos.X + boxW / 2, boxPos.Y + screenH + 4)
        d.Dist.Color    = GetESPColor(player, "Distance")
        d.Dist.Size     = ESPSettings.TextSize - 2
        d.Dist.Visible  = true
    else
        d.Dist.Visible = false
    end

    -- ── Snaplines ────────────────────────────
    if ESPSettings.Snaplines then
        d.Snap.From    = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y)
        d.Snap.To      = Vector2.new(pos.X, pos.Y)
        d.Snap.Color   = GetESPColor(player, "Snapline")
        d.Snap.Visible = true
    else
        d.Snap.Visible = false
    end

    -- ── Chams ────────────────────────────────
    local hl = ESPHighlight[player]
    if hl then
        if ESPSettings.ChamsEnabled and char then
            hl.Parent                = char
            hl.FillColor             = ESPColors.ChamsFill
            hl.OutlineColor          = ESPColors.ChamsOutline
            hl.FillTransparency      = ESPSettings.ChamsTransparency
            hl.OutlineTransparency   = ESPSettings.ChamsOutlineTransparency
            hl.Enabled               = true
        else
            hl.Enabled = false
        end
    end

    -- ── Skeleton ─────────────────────────────
    if ESPSettings.SkeletonESP then
        local sk = ESPSkeleton[player]
        if sk and char then
            local function F(n) return char:FindFirstChild(n) end
            local head = F("Head")
            local utor = F("UpperTorso") or F("Torso")
            local ltor = F("LowerTorso") or F("Torso")
            local lUA  = F("LeftUpperArm")  or F("Left Arm")
            local lLA  = F("LeftLowerArm")  or F("Left Arm")
            local lH   = F("LeftHand")      or F("Left Arm")
            local rUA  = F("RightUpperArm") or F("Right Arm")
            local rLA  = F("RightLowerArm") or F("Right Arm")
            local rH   = F("RightHand")     or F("Right Arm")
            local lUL  = F("LeftUpperLeg")  or F("Left Leg")
            local lLL  = F("LeftLowerLeg")  or F("Left Leg")
            local lF   = F("LeftFoot")      or F("Left Leg")
            local rUL  = F("RightUpperLeg") or F("Right Leg")
            local rLL  = F("RightLowerLeg") or F("Right Leg")
            local rF   = F("RightFoot")     or F("Right Leg")

            DrawBone(head, utor, sk.Head,       player)
            DrawBone(utor, ltor, sk.UpperSpine, player)
            DrawBone(utor, lUA,  sk.LShoulder,  player)
            DrawBone(lUA,  lLA,  sk.LUpperArm,  player)
            DrawBone(lLA,  lH,   sk.LLowerArm,  player)
            DrawBone(utor, rUA,  sk.RShoulder,  player)
            DrawBone(rUA,  rLA,  sk.RUpperArm,  player)
            DrawBone(rLA,  rH,   sk.RLowerArm,  player)
            DrawBone(ltor, lUL,  sk.LHip,       player)
            DrawBone(lUL,  lLL,  sk.LUpperLeg,  player)
            DrawBone(lLL,  lF,   sk.LLowerLeg,  player)
            DrawBone(ltor, rUL,  sk.RHip,       player)
            DrawBone(rUL,  rLL,  sk.RUpperLeg,  player)
            DrawBone(rLL,  rF,   sk.RLowerLeg,  player)
        end
    else
        local sk = ESPSkeleton[player]
        if sk then for _, l in pairs(sk) do l.Visible = false end end
    end
end

-- Init ESP for current players
for _, p in pairs(Players:GetPlayers()) do CreateESPEntry(p) end
Players.PlayerAdded:Connect(CreateESPEntry)
Players.PlayerRemoving:Connect(RemoveESPEntry)

-- ──────────────────────────────────────────────
--  RAINBOW  —  COLOR CYCLE
-- ──────────────────────────────────────────────

task.spawn(function()
    while task.wait(0.1) do
        ESPColors.Rainbow = Color3.fromHSV(tick() * ESPSettings.RainbowSpeed % 1, 1, 1)
    end
end)

-- ──────────────────────────────────────────────
--  MAIN RENDER LOOP
-- ──────────────────────────────────────────────

local lastESPUpdate = 0

RunService.RenderStepped:Connect(function()
    -- FOV circle
    Circle.Position = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
    Circle.Radius   = FOV
    Circle.Visible  = ShowFOV
    Circle.Color    = FOVColor

    -- Target lock
    if TargetLock then
        local target = GetClosest()
        if target and target.Character and target.Character:FindFirstChild("Head") then
            local goal = CFrame.new(Camera.CFrame.Position, target.Character.Head.Position)
            Camera.CFrame = Camera.CFrame:Lerp(goal, SmoothAmount)
        end
    end

    -- ESP (rate-limited to 144 hz)
    local now = tick()
    if now - lastESPUpdate >= 1 / 144 then
        for _, p in pairs(Players:GetPlayers()) do
            if p ~= LocalPlayer then
                if not ESPDrawings[p] then CreateESPEntry(p) end
                UpdateESPEntry(p)
            end
        end
        lastESPUpdate = now
    end

    -- Speed / Jump
    local char = LocalPlayer.Character
    if not char then return end
    local hum = char:FindFirstChildOfClass("Humanoid")
    if hum then
        hum.WalkSpeed    = SpeedEnabled and SpeedValue or 16
        hum.UseJumpPower = true
        hum.JumpPower    = JumpEnabled and JumpValue or 50
    end
end)

-- ──────────────────────────────────────────────
--  INFINITE JUMP  /  BUNNY HOP
-- ──────────────────────────────────────────────

UIS.JumpRequest:Connect(function()
    if InfJump then
        local hum = GetChar():FindFirstChildOfClass("Humanoid")
        if hum then hum:ChangeState(Enum.HumanoidStateType.Jumping) end
    end

    if BunnyHop then
        local c = LocalPlayer.Character
        if not c then return end
        local h = c:FindFirstChildOfClass("Humanoid")
        local r = c:FindFirstChild("HumanoidRootPart")
        if h and r then
            h:ChangeState(Enum.HumanoidStateType.Jumping)
            r.Velocity = Vector3.new(r.Velocity.X, 50, r.Velocity.Z)
        end
    end
end)

-- ══════════════════════════════════════════════
--  UI  —  COMBAT TAB
-- ══════════════════════════════════════════════

CombatTab:CreateSection("🎯 Aim")

CombatTab:CreateToggle({
    Name         = "🎯 Target Lock",
    Flag         = "TargetLock",
    CurrentValue = false,
    Callback     = function(v) TargetLock = v end,
})
CombatTab:CreateSlider({
    Name         = "Smooth Amount",
    Flag         = "SmoothAmount",
    Range        = { 1, 100 },
    Increment    = 1,
    CurrentValue = 15,
    Callback     = function(v) SmoothAmount = v / 100 end,
})

CombatTab:CreateSection("🔵 FOV")

CombatTab:CreateSlider({
    Name         = "FOV Size",
    Flag         = "FOVSize",
    Range        = { 0, 400 },
    Increment    = 1,
    CurrentValue = 150,
    Callback     = function(v) FOV = v end,
})
CombatTab:CreateToggle({
    Name         = "Show FOV Circle",
    Flag         = "ShowFOV",
    CurrentValue = true,
    Callback     = function(v) ShowFOV = v end,
})
CombatTab:CreateColorPicker({
    Name     = "FOV Circle Color",
    Flag     = "FOVColor",
    Color    = FOVColor,
    Callback = function(v) FOVColor = v; SaveColors() end,
})

-- ══════════════════════════════════════════════
--  UI  —  VISUAL TAB
-- ══════════════════════════════════════════════

VisualTab:CreateSection("👁️ General")

VisualTab:CreateToggle({
    Name         = "👁️ Enable ESP",
    Flag         = "ESPEnabled",
    CurrentValue = false,
    Callback     = function(v) ESPSettings.Enabled = v end,
})
VisualTab:CreateToggle({
    Name         = "🫂 Team Check",
    Flag         = "ESPTeamCheck",
    CurrentValue = true,
    Callback     = function(v) ESPSettings.TeamCheck = v; TeamCheck = v end,
})
VisualTab:CreateSlider({
    Name         = "Max Distance",
    Flag         = "ESPMaxDist",
    Range        = { 100, 5000 },
    Increment    = 100,
    CurrentValue = 1000,
    Callback     = function(v) ESPSettings.MaxDistance = v end,
})

VisualTab:CreateDivider()
VisualTab:CreateSection("📦 Box")

VisualTab:CreateToggle({
    Name         = "📦 Box ESP",
    Flag         = "BoxESP",
    CurrentValue = false,
    Callback     = function(v) ESPSettings.BoxESP = v end,
})
VisualTab:CreateDropdown({
    Name          = "Box Style",
    Flag          = "BoxStyle",
    Options       = { "Corner", "Full" },
    CurrentOption = { "Corner" },
    Callback      = function(v) ESPSettings.BoxStyle = v[1] end,
})
VisualTab:CreateSlider({
    Name         = "Box Thickness",
    Flag         = "BoxThickness",
    Range        = { 1, 5 },
    Increment    = 1,
    CurrentValue = 1,
    Callback     = function(v) ESPSettings.BoxThickness = v end,
})
VisualTab:CreateColorPicker({
    Name     = "Box — Enemy",
    Flag     = "BoxColorEnemy",
    Color    = ESPColors.BoxEnemy,
    Callback = function(v) ESPColors.BoxEnemy = v; SaveColors() end,
})
VisualTab:CreateColorPicker({
    Name     = "Box — Ally",
    Flag     = "BoxColorAlly",
    Color    = ESPColors.BoxAlly,
    Callback = function(v) ESPColors.BoxAlly = v; SaveColors() end,
})

VisualTab:CreateDivider()
VisualTab:CreateSection("📡 Tracer")

VisualTab:CreateToggle({
    Name         = "📡 Tracer ESP",
    Flag         = "TracerESP",
    CurrentValue = false,
    Callback     = function(v) ESPSettings.TracerESP = v end,
})
VisualTab:CreateDropdown({
    Name          = "Tracer Origin",
    Flag          = "TracerOrigin",
    Options       = { "Bottom", "Top", "Mouse", "Center" },
    CurrentOption = { "Bottom" },
    Callback      = function(v) ESPSettings.TracerOrigin = v[1] end,
})
VisualTab:CreateSlider({
    Name         = "Tracer Thickness",
    Flag         = "TracerThickness",
    Range        = { 1, 5 },
    Increment    = 1,
    CurrentValue = 1,
    Callback     = function(v) ESPSettings.TracerThickness = v end,
})
VisualTab:CreateColorPicker({
    Name     = "Tracer — Enemy",
    Flag     = "TracerColorEnemy",
    Color    = ESPColors.TracerEnemy,
    Callback = function(v) ESPColors.TracerEnemy = v; SaveColors() end,
})
VisualTab:CreateColorPicker({
    Name     = "Tracer — Ally",
    Flag     = "TracerColorAlly",
    Color    = ESPColors.TracerAlly,
    Callback = function(v) ESPColors.TracerAlly = v; SaveColors() end,
})

VisualTab:CreateDivider()
VisualTab:CreateSection("❤️ Health")

VisualTab:CreateToggle({
    Name         = "❤️ Health ESP",
    Flag         = "HealthESP",
    CurrentValue = false,
    Callback     = function(v) ESPSettings.HealthESP = v end,
})
VisualTab:CreateDropdown({
    Name          = "Health Style",
    Flag          = "HealthStyle",
    Options       = { "Bar", "Text", "Both" },
    CurrentOption = { "Bar" },
    Callback      = function(v) ESPSettings.HealthStyle = v[1] end,
})
VisualTab:CreateColorPicker({
    Name     = "Health Bar Color",
    Flag     = "HealthBarColor",
    Color    = ESPColors.HealthBar,
    Callback = function(v) ESPColors.HealthBar = v; SaveColors() end,
})
VisualTab:CreateColorPicker({
    Name     = "Health Text Color",
    Flag     = "HealthTextColor",
    Color    = ESPColors.HealthText,
    Callback = function(v) ESPColors.HealthText = v; SaveColors() end,
})

VisualTab:CreateDivider()
VisualTab:CreateSection("🏷️ Name & Distance")

VisualTab:CreateToggle({
    Name         = "🏷️ Name ESP",
    Flag         = "NameESP",
    CurrentValue = false,
    Callback     = function(v) ESPSettings.NameESP = v end,
})
VisualTab:CreateColorPicker({
    Name     = "Name — Enemy",
    Flag     = "NameColorEnemy",
    Color    = ESPColors.NameEnemy,
    Callback = function(v) ESPColors.NameEnemy = v; SaveColors() end,
})
VisualTab:CreateColorPicker({
    Name     = "Name — Ally",
    Flag     = "NameColorAlly",
    Color    = ESPColors.NameAlly,
    Callback = function(v) ESPColors.NameAlly = v; SaveColors() end,
})
VisualTab:CreateToggle({
    Name         = "📏 Show Distance",
    Flag         = "ShowDistance",
    CurrentValue = true,
    Callback     = function(v) ESPSettings.ShowDistance = v end,
})
VisualTab:CreateColorPicker({
    Name     = "Distance Color",
    Flag     = "DistanceColor",
    Color    = ESPColors.Distance,
    Callback = function(v) ESPColors.Distance = v; SaveColors() end,
})

VisualTab:CreateDivider()
VisualTab:CreateSection("📌 Snaplines")

VisualTab:CreateToggle({
    Name         = "📌 Snaplines",
    Flag         = "Snaplines",
    CurrentValue = false,
    Callback     = function(v) ESPSettings.Snaplines = v end,
})
VisualTab:CreateColorPicker({
    Name     = "Snapline — Enemy",
    Flag     = "SnapColorEnemy",
    Color    = ESPColors.SnaplineEnemy,
    Callback = function(v) ESPColors.SnaplineEnemy = v; SaveColors() end,
})
VisualTab:CreateColorPicker({
    Name     = "Snapline — Ally",
    Flag     = "SnapColorAlly",
    Color    = ESPColors.SnaplineAlly,
    Callback = function(v) ESPColors.SnaplineAlly = v; SaveColors() end,
})

VisualTab:CreateDivider()
VisualTab:CreateSection("🦴 Skeleton")

VisualTab:CreateToggle({
    Name         = "🦴 Skeleton ESP",
    Flag         = "SkeletonESP",
    CurrentValue = false,
    Callback     = function(v) ESPSettings.SkeletonESP = v end,
})
VisualTab:CreateSlider({
    Name         = "Skeleton Thickness",
    Flag         = "SkeletonThickness",
    Range        = { 1, 5 },
    Increment    = 1,
    CurrentValue = 1,
    Callback     = function(v) ESPSettings.SkeletonThickness = v end,
})
VisualTab:CreateColorPicker({
    Name     = "Skeleton Color",
    Flag     = "SkeletonColor",
    Color    = ESPColors.Skeleton,
    Callback = function(v) ESPColors.Skeleton = v; SaveColors() end,
})

VisualTab:CreateDivider()
VisualTab:CreateSection("🎨 Chams")

VisualTab:CreateToggle({
    Name         = "🎨 Chams",
    Flag         = "ChamsEnabled",
    CurrentValue = false,
    Callback     = function(v) ESPSettings.ChamsEnabled = v end,
})
VisualTab:CreateSlider({
    Name         = "Fill Transparency",
    Flag         = "ChamsTransparency",
    Range        = { 0, 100 },
    Increment    = 1,
    CurrentValue = 50,
    Callback     = function(v) ESPSettings.ChamsTransparency = v / 100 end,
})
VisualTab:CreateColorPicker({
    Name     = "Chams — Fill",
    Flag     = "ChamsFillColor",
    Color    = ESPColors.ChamsFill,
    Callback = function(v) ESPColors.ChamsFill = v; SaveColors() end,
})
VisualTab:CreateColorPicker({
    Name     = "Chams — Outline",
    Flag     = "ChamsOutlineColor",
    Color    = ESPColors.ChamsOutline,
    Callback = function(v) ESPColors.ChamsOutline = v; SaveColors() end,
})

VisualTab:CreateDivider()
VisualTab:CreateSection("🌈 Rainbow")

VisualTab:CreateToggle({
    Name         = "🌈 Rainbow Mode",
    Flag         = "RainbowMode",
    CurrentValue = false,
    Callback     = function(v) ESPSettings.RainbowEnabled = v end,
})
VisualTab:CreateSlider({
    Name         = "Rainbow Speed",
    Flag         = "RainbowSpeed",
    Range        = { 1, 10 },
    Increment    = 1,
    CurrentValue = 1,
    Callback     = function(v) ESPSettings.RainbowSpeed = v end,
})

-- ══════════════════════════════════════════════
--  UI  —  PLAYER TAB
-- ══════════════════════════════════════════════

PlayerTab:CreateSection("⚡ Speed")

PlayerTab:CreateToggle({
    Name         = "⚡ Speed Hack",
    Flag         = "SpeedEnabled",
    CurrentValue = false,
    Callback     = function(v) SpeedEnabled = v end,
})
PlayerTab:CreateSlider({
    Name         = "Speed Value",
    Flag         = "SpeedValue",
    Range        = { 16, 500 },
    Increment    = 1,
    CurrentValue = 50,
    Callback     = function(v) SpeedValue = v end,
})

PlayerTab:CreateDivider()
PlayerTab:CreateSection("🦘 Jump")

PlayerTab:CreateToggle({
    Name         = "🦘 Jump Boost",
    Flag         = "JumpEnabled",
    CurrentValue = false,
    Callback     = function(v) JumpEnabled = v end,
})
PlayerTab:CreateSlider({
    Name         = "Jump Power",
    Flag         = "JumpValue",
    Range        = { 50, 500 },
    Increment    = 1,
    CurrentValue = 120,
    Callback     = function(v) JumpValue = v end,
})
PlayerTab:CreateToggle({
    Name         = "♾️ Infinite Jump",
    Flag         = "InfJump",
    CurrentValue = false,
    Callback     = function(v) InfJump = v end,
})
PlayerTab:CreateToggle({
    Name         = "🐇 Bunny Hop",
    Flag         = "BunnyHop",
    CurrentValue = false,
    Callback     = function(v) BunnyHop = v end,
})

PlayerTab:CreateDivider()
PlayerTab:CreateSection("🌀 Misc Movement")

PlayerTab:CreateToggle({
    Name         = "👻 NoClip",
    Flag         = "NoClip",
    CurrentValue = false,
    Callback     = function(v) NoClip = v end,
})
PlayerTab:CreateButton({
    Name     = "🛩️ Fly GUI V3",
    Callback = function()
        loadstring(game:HttpGet("https://raw.githubusercontent.com/XNEOFF/FlyGuiV3/main/FlyGuiV3.txt"))()
    end,
})

-- ══════════════════════════════════════════════
--  UI  —  MISC TAB
-- ══════════════════════════════════════════════

MiscTab:CreateSection("🌟 Environment")

MiscTab:CreateToggle({
    Name         = "🌟 Fullbright",
    Flag         = "Fullbright",
    CurrentValue = false,
    Callback     = function(v)
        Lighting.Brightness    = v and 10 or 1
        Lighting.ClockTime     = 14
        Lighting.FogEnd        = v and 1000000 or 100000
        Lighting.GlobalShadows = not v
    end,
})

MiscTab:CreateDivider()
MiscTab:CreateSection("🚀 Performance")

MiscTab:CreateToggle({
    Name         = "🚀 AntiLag",
    Flag         = "AntiLag",
    CurrentValue = false,
    Callback     = function(v)
        AntiLagEnabled = v
        if v then ApplyAntiLag() else DisableAntiLag() end
    end,
})

MiscTab:CreateDivider()
MiscTab:CreateSection("📋 About")

MiscTab:CreateLabel("Didi Hub V3  |  by 46dgin  |  v3.0")
MiscTab:CreateButton({
    Name     = "🔔 Credits",
    Callback = function()
        Rayfield:Notify({
            Title    = "Didi Hub V3",
            Content  = "Developed by 46dgin\nVersion 3.0",
            Duration = 5,
        })
    end,
})

-- ──────────────────────────────────────────────
--  LOAD PROFILE  +  WELCOME
-- ──────────────────────────────────────────────

LoadColors()
Rayfield:LoadConfiguration()

Rayfield:Notify({
    Title    = "Didi Hub V3",
    Content  = "Loaded successfully ✅  |  ESP → Visual tab",
    Duration = 5,
})
