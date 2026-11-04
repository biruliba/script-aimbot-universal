-- ============================================================
-- DANNY SCRIPT — ADVANCED AIM + ESP v2.8
-- Autor: Danny | Xeno-compatible | Lua / Roblox
-- ============================================================

local AUTHOR = "Danny"
local SCRIPT_NAME = "Danny Script"
local SCRIPT_VERSION = "2.8"
local BUBBLE_IMAGE = "rbxthumb://type=Asset&id=100150312706947&w=150&h=150"

local SOCIAL = {
    youtube = "https://www.youtube.com/@dehmencia",
    tiktok  = "https://www.tiktok.com/@dehmencia?lang=pt-BR",
}

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")
local HttpService = game:GetService("HttpService")
local Lighting = game:GetService("Lighting")
local TweenService = game:GetService("TweenService")
local lp = Players.LocalPlayer
local cam = workspace.CurrentCamera

-- ============================================================
-- CONFIG
-- ============================================================
local DEFAULTS = {
    meta = { author = AUTHOR, name = SCRIPT_NAME, version = SCRIPT_VERSION },
    aimbot = {
        enabled = false, key = Enum.KeyCode.E, fov = 180, smooth = 0.18,
        distance = 500, prediction = 0, priority = "Distance", lock = false,
        part = "Head", showFov = true, fovColor = Color3.fromRGB(0, 200, 255),
        fovThickness = 1, predictionEnabled = false,
    },
    filters = {
        teamCheck = true, ignoreDead = true, ignoreInvisible = true,
        ignoreNpcs = true, ignoreNoTeam = false, onlyLowHealth = false,
        lowHealthThreshold = 30,
    },
    esp = {
        enabled = false, box = false, cornerBox = true, skeleton = false,
        head = false, name = true, distance = true, health = true,
        armor = false, weapon = false, team = true, line = false, glow = true,
        enemyColor = Color3.fromRGB(255, 60, 60),
        allyColor = Color3.fromRGB(60, 160, 255),
        npcColor = Color3.fromRGB(220, 200, 60),
        transparency = 0.55, lineThickness = 1, textSize = 12,
        maxDistance = 1000, showEnemies = true, showAllies = true,
        healthBar = true, healthBarWidth = 4,
    },
    team = {
        enabled = true, showTeamName = true, highlightEnemies = true,
        teamColors = {},
    },
    visuals = { fullbright = false, noclip = false },
    ui = {
        scale = 1, transparency = 0, bubblePos = UDim2.new(0, 40, 0, 40),
        menuPos = UDim2.new(0.5, -360, 0.5, -240),
        stateBubblePos = UDim2.new(0, 40, 0, 100),
    },
    profiles = { active = "default", saved = {} },
}

local function deepCopy(t)
    if type(t) ~= "table" then return t end
    local r = {}
    for k, v in pairs(t) do r[k] = deepCopy(v) end
    return r
end

local function serializeColor(c) return {R = c.R, G = c.G, B = c.B} end
local function deserializeColor(t)
    if type(t) == "table" and t.R then return Color3.new(t.R, t.G, t.B) end
    return Color3.new(1,1,1)
end
local function serializeUDim2(u) return {XS = u.X.Scale, XO = u.X.Offset, YS = u.Y.Scale, YO = u.Y.Offset} end
local function deserializeUDim2(t)
    if type(t) == "table" and t.XS then return UDim2.new(t.XS, t.XO, t.YS, t.YO) end
    return UDim2.new(0, 40, 0, 40)
end

local function encodeConfig(cfg)
    local out = {}
    for section, values in pairs(cfg) do
        out[section] = {}
        for k, v in pairs(values) do
            if typeof(v) == "Color3" then
                out[section][k] = {__type = "Color3", data = serializeColor(v)}
            elseif typeof(v) == "UDim2" then
                out[section][k] = {__type = "UDim2", data = serializeUDim2(v)}
            elseif typeof(v) == "EnumItem" then
                out[section][k] = {__type = "Enum", enum = tostring(v.EnumType), name = v.Name}
            else
                out[section][k] = v
            end
        end
    end
    return HttpService:JSONEncode(out)
end

local function decodeConfig(str)
    local ok, decoded = pcall(HttpService.JSONDecode, HttpService, str)
    if not ok then return nil end
    for section, values in pairs(decoded) do
        if type(values) == "table" then
            for k, v in pairs(values) do
                if type(v) == "table" and v.__type == "Color3" then
                    decoded[section][k] = deserializeColor(v.data)
                elseif type(v) == "table" and v.__type == "UDim2" then
                    decoded[section][k] = deserializeUDim2(v.data)
                elseif type(v) == "table" and v.__type == "Enum" then
                    local ok2, enum = pcall(function() return Enum[v.enum][v.name] end)
                    if ok2 then decoded[section][k] = enum end
                end
            end
        end
    end
    return decoded
end

local Config = { data = deepCopy(DEFAULTS) }

function Config.save()
    pcall(function()
        if writefile then writefile("danny_script_v28.json", encodeConfig(Config.data)) end
    end)
end

function Config.load()
    local ok, raw = pcall(function()
        if isfile and isfile("danny_script_v28.json") then return readfile("danny_script_v28.json") end
    end)
    if ok and raw then
        local d = decodeConfig(raw)
        if d then for k, v in pairs(d) do Config.data[k] = v end; return true end
    end
    return false
end

function Config.reset()
    Config.data = deepCopy(DEFAULTS)
    Config.data.aimbot.key = Enum.KeyCode.E
    Config.save()
end

function Config.export() return encodeConfig(Config.data) end
function Config.import(str)
    local d = decodeConfig(str)
    if d then Config.data = d; Config.save(); return true end
    return false
end

Config.load()

-- ============================================================
-- THEME
-- ============================================================
local Theme = {
    bg = Color3.fromRGB(16, 16, 22), panel = Color3.fromRGB(24, 24, 32),
    panel2 = Color3.fromRGB(34, 34, 46), panel3 = Color3.fromRGB(44, 44, 58),
    accent = Color3.fromRGB(0, 200, 255), accent2 = Color3.fromRGB(120, 80, 255),
    text = Color3.fromRGB(230, 230, 245), textDim = Color3.fromRGB(150, 150, 175),
    ok = Color3.fromRGB(80, 220, 120), bad = Color3.fromRGB(255, 80, 80),
    warn = Color3.fromRGB(255, 200, 60),
    font = Enum.Font.Gotham, fontBold = Enum.Font.GothamBold,
}
local function corner(p, r) local c = Instance.new("UICorner"); c.CornerRadius = UDim.new(0, r or 8); c.Parent = p; return c end
local function stroke(p, color, thickness) local s = Instance.new("UIStroke"); s.Color = color or Theme.panel2; s.Thickness = thickness or 1; s.Parent = p; return s end
local function pad(p, t, l, r, b)
    local u = Instance.new("UIPadding")
    if t then u.PaddingTop = UDim.new(0, t) end
    if l then u.PaddingLeft = UDim.new(0, l) end
    if r then u.PaddingRight = UDim.new(0, r) end
    if b then u.PaddingBottom = UDim.new(0, b) end
    u.Parent = p; return u
end
local function label(p, text, size, color, bold)
    local l = Instance.new("TextLabel")
    l.BackgroundTransparency = 1; l.Text = text; l.TextSize = size or 13
    l.TextColor3 = color or Theme.text; l.Font = bold and Theme.fontBold or Theme.font
    l.TextXAlignment = Enum.TextXAlignment.Left; l.Parent = p; return l
end

-- ============================================================
-- GUI PRINCIPAL
-- ============================================================
local gui = Instance.new("ScreenGui")
gui.Name = SCRIPT_NAME .. "_v" .. SCRIPT_VERSION
gui.ResetOnSpawn = false
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.IgnoreGuiInset = true
gui.Parent = CoreGui

local main = Instance.new("Frame")
main.Size = UDim2.new(0, 720, 0, 480)
main.Position = Config.data.ui.menuPos
main.BackgroundColor3 = Theme.bg; main.BorderSizePixel = 0
main.Active = true; main.ClipsDescendants = true; main.Parent = gui
corner(main, 12); stroke(main, Theme.panel2, 1)

local bar = Instance.new("Frame")
bar.Size = UDim2.new(1, 0, 0, 44); bar.BackgroundColor3 = Theme.panel; bar.Parent = main
corner(bar, 12)

local ttl = label(bar, "  ⚡ DANNY SCRIPT v" .. SCRIPT_VERSION, 15, Theme.accent, true)
ttl.Size = UDim2.new(0.5, 0, 1, 0); ttl.Position = UDim2.new(0, 14, 0, 0)

local authorTag = label(bar, "by " .. AUTHOR, 10, Theme.textDim, false)
authorTag.Size = UDim2.new(0.2, 0, 1, 0)
authorTag.Position = UDim2.new(0.32, 0, 0, 0)

local statusDot = Instance.new("Frame")
statusDot.Size = UDim2.new(0, 8, 0, 8)
statusDot.Position = UDim2.new(0, 220, 0.5, -4)
statusDot.BackgroundColor3 = Theme.bad; statusDot.Parent = bar
corner(statusDot, 4)

local function winBtn(txt, x, color, fn)
    local b = Instance.new("TextButton")
    b.Size = UDim2.new(0, 28, 0, 28); b.Position = UDim2.new(1, x, 0.5, -14)
    b.BackgroundColor3 = color; b.Text = txt; b.TextColor3 = Color3.new(1,1,1)
    b.Font = Theme.fontBold; b.TextSize = 14; b.Parent = bar
    corner(b, 6); b.MouseButton1Click:Connect(fn)
end
winBtn("–", -108, Theme.panel2, function() main.Visible = false end)
winBtn("□", -76, Theme.panel2, function()
    main.Size = UDim2.new(0, 720, 0, 480); main.Position = UDim2.new(0.5, -360, 0.5, -240)
end)
winBtn("×", -44, Theme.bad, function() main.Visible = false end)

local search = Instance.new("TextBox")
search.Size = UDim2.new(0, 200, 0, 28); search.Position = UDim2.new(1, -330, 0.5, -14)
search.BackgroundColor3 = Theme.panel2; search.BorderSizePixel = 0
search.PlaceholderText = "Buscar configuração..."; search.Text = ""
search.TextColor3 = Theme.text; search.PlaceholderColor3 = Theme.textDim
search.Font = Theme.font; search.TextSize = 12; search.Parent = bar
corner(search, 6); pad(search, 0, 8, 0, 0)

local side = Instance.new("Frame")
side.Size = UDim2.new(0, 160, 1, -44); side.Position = UDim2.new(0, 0, 0, 44)
side.BackgroundColor3 = Theme.panel; side.BorderSizePixel = 0; side.Parent = main
local sideList = Instance.new("UIListLayout"); sideList.Padding = UDim.new(0, 2); sideList.Parent = side
pad(side, 8, 6, 6, 8)

local UI = { frames = {}, activeTab = "Home" }
local TABS = {"Home","Aimbot","ESP","Team","Players","Visuals","Settings","Profiles","About"}
local tabButtons = {}

for _, name in ipairs(TABS) do
    local tb = Instance.new("TextButton")
    tb.Size = UDim2.new(1, 0, 0, 32); tb.BackgroundColor3 = Theme.panel
    tb.Text = "  " .. name; tb.TextColor3 = Theme.textDim
    tb.Font = Theme.font; tb.TextSize = 13; tb.TextXAlignment = Enum.TextXAlignment.Left
    tb.Parent = side; corner(tb, 6); tabButtons[name] = tb

    local content = Instance.new("Frame")
    content.Size = UDim2.new(1, -170, 1, -54); content.Position = UDim2.new(0, 160, 0, 44)
    content.BackgroundTransparency = 1; content.Visible = (name == "Home"); content.Parent = main

    local scroll = Instance.new("ScrollingFrame")
    scroll.Size = UDim2.new(1, -10, 1, -10); scroll.Position = UDim2.new(0, 5, 0, 5)
    scroll.BackgroundTransparency = 1; scroll.BorderSizePixel = 0
    scroll.ScrollBarThickness = 4; scroll.ScrollBarImageColor3 = Theme.accent
    scroll.Parent = content
    local sl = Instance.new("UIListLayout"); sl.Padding = UDim.new(0, 6)
    sl.SortOrder = Enum.SortOrder.LayoutOrder; sl.Parent = scroll
    sl:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
        scroll.CanvasSize = UDim2.new(0, 0, 0, sl.AbsoluteContentSize.Y + 10)
    end)

    UI.frames[name] = {content = content, scroll = scroll}
    tb.MouseButton1Click:Connect(function()
        for n, f in pairs(UI.frames) do f.content.Visible = (n == name) end
        for n, b in pairs(tabButtons) do
            b.BackgroundColor3 = (n == name) and Theme.panel2 or Theme.panel
            b.TextColor3 = (n == name) and Theme.accent or Theme.textDim
        end
        UI.activeTab = name
    end)
end

-- ============================================================
-- WIDGETS
-- ============================================================
local function section(tab, title)
    local f = Instance.new("Frame")
    f.Size = UDim2.new(1, 0, 0, 26); f.BackgroundColor3 = Theme.panel2
    f.Parent = UI.frames[tab].scroll; corner(f, 6)
    label(f, "  " .. title, 11, Theme.accent, true).Size = UDim2.new(1, 0, 1, 0)
end

local function toggle(tab, text, init, cb)
    local b = Instance.new("TextButton")
    b.Size = UDim2.new(1, 0, 0, 32); b.BackgroundColor3 = Theme.panel
    b.Text = ""; b.Parent = UI.frames[tab].scroll; corner(b, 6); stroke(b, Theme.panel2)
    b.Name = text
    label(b, "  " .. text, 13, Theme.text).Size = UDim2.new(0.75, 0, 1, 0)
    local dot = Instance.new("Frame")
    dot.Size = UDim2.new(0, 18, 0, 18); dot.Position = UDim2.new(1, -28, 0.5, -9)
    dot.BackgroundColor3 = init and Theme.ok or Theme.panel2; dot.Parent = b; corner(dot, 9)
    local state = init
    b.MouseButton1Click:Connect(function()
        state = not state
        dot.BackgroundColor3 = state and Theme.ok or Theme.panel2
        cb(state); Config.save()
    end)
end

local function slider(tab, text, min, max, init, cb)
    local row = Instance.new("Frame")
    row.Size = UDim2.new(1, 0, 0, 46); row.BackgroundColor3 = Theme.panel
    row.Parent = UI.frames[tab].scroll; corner(row, 6); stroke(row, Theme.panel2)
    row.Name = text
    local l = label(row, "  " .. text, 12, Theme.text); l.Size = UDim2.new(0.6, 0, 0, 22)
    local val = label(row, tostring(init), 12, Theme.accent, true)
    val.Size = UDim2.new(0.3, 0, 0, 22); val.Position = UDim2.new(0.65, 0, 0, 0)
    val.TextXAlignment = Enum.TextXAlignment.Right
    local track = Instance.new("Frame")
    track.Size = UDim2.new(1, -24, 0, 4); track.Position = UDim2.new(0, 12, 0, 34)
    track.BackgroundColor3 = Theme.panel3; track.Parent = row; corner(track, 2)
    local fill = Instance.new("Frame")
    local pct = (init - min) / (max - min)
    fill.Size = UDim2.new(pct, 0, 1, 0); fill.BackgroundColor3 = Theme.accent
    fill.Parent = track; corner(fill, 2)
    local dragging = false
    track.InputBegan:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 then dragging = true end end)
    UserInputService.InputEnded:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end end)
    UserInputService.InputChanged:Connect(function(i)
        if dragging and i.UserInputType == Enum.UserInputType.MouseMovement then
            local rel = math.clamp((i.Position.X - track.AbsolutePosition.X) / track.AbsoluteSize.X, 0, 1)
            fill.Size = UDim2.new(rel, 0, 1, 0)
            local v = min + (max - min) * rel
            v = (max - min) >= 10 and math.floor(v) or math.floor(v * 100) / 100
            val.Text = tostring(v); cb(v)
        end
    end)
end

local function keybind(tab, text, initial, cb)
    local row = Instance.new("Frame")
    row.Size = UDim2.new(1, 0, 0, 32); row.BackgroundColor3 = Theme.panel
    row.Parent = UI.frames[tab].scroll; corner(row, 6); stroke(row, Theme.panel2); row.Name = text
    label(row, "  " .. text, 13, Theme.text).Size = UDim2.new(0.55, 0, 1, 0)
    local b = Instance.new("TextButton")
    b.Size = UDim2.new(0, 110, 0, 22); b.Position = UDim2.new(1, -122, 0.5, -11)
    b.BackgroundColor3 = Theme.panel2; b.Text = initial.Name; b.TextColor3 = Theme.text
    b.Font = Theme.fontBold; b.TextSize = 12; b.Parent = row; corner(b, 5)
    local listening = false
    b.MouseButton1Click:Connect(function()
        if listening then return end
        listening = true; b.Text = "..."
        local conn
        conn = UserInputService.InputBegan:Connect(function(i, gp)
            if gp then return end
            if i.UserInputType == Enum.UserInputType.Keyboard then
                b.Text = i.KeyCode.Name; listening = false; conn:Disconnect()
                cb(i.KeyCode); Config.save()
            end
        end)
    end)
end

local function dropdown(tab, text, options, initial, cb)
    local row = Instance.new("Frame")
    row.Size = UDim2.new(1, 0, 0, 32); row.BackgroundColor3 = Theme.panel
    row.Parent = UI.frames[tab].scroll; corner(row, 6); stroke(row, Theme.panel2); row.Name = text
    label(row, "  " .. text, 13, Theme.text).Size = UDim2.new(0.5, 0, 1, 0)
    local b = Instance.new("TextButton")
    b.Size = UDim2.new(0, 150, 0, 22); b.Position = UDim2.new(1, -162, 0.5, -11)
    b.BackgroundColor3 = Theme.panel2; b.Text = initial; b.TextColor3 = Theme.accent
    b.Font = Theme.fontBold; b.TextSize = 12; b.Parent = row; corner(b, 5)
    local idx = 1
    for i, o in ipairs(options) do if o == initial then idx = i end end
    b.MouseButton1Click:Connect(function()
        idx = idx % #options + 1; b.Text = options[idx]; cb(options[idx]); Config.save()
    end)
end

local function button(tab, text, color, cb)
    local b = Instance.new("TextButton")
    b.Size = UDim2.new(1, 0, 0, 32); b.BackgroundColor3 = color or Theme.panel2
    b.Text = text; b.TextColor3 = Color3.new(1,1,1); b.Font = Theme.fontBold
    b.TextSize = 13; b.Parent = UI.frames[tab].scroll; corner(b, 6)
    b.MouseButton1Click:Connect(cb)
end

-- ============================================================
-- ESTADO
-- ============================================================
local updateBubbleLook

local function updateStatus()
    statusDot.BackgroundColor3 = Config.data.aimbot.enabled and Theme.ok or Theme.bad
    if updateBubbleLook then updateBubbleLook() end
end

-- ============================================================
-- ABAS (conteúdo)
-- ============================================================
section("Home", "STATUS")
local statusLabel = label(UI.frames["Home"].scroll, "  Aimbot: OFF  •  ESP: OFF", 12, Theme.textDim)
statusLabel.Size = UDim2.new(1, 0, 0, 24)
section("Home", "AÇÕES RÁPIDAS")
button("Home", "⚡ Toggle Aimbot", Theme.panel2, function()
    Config.data.aimbot.enabled = not Config.data.aimbot.enabled
    Config.save(); updateStatus(); if _G.updateStateBubble then _G.updateStateBubble() end
end)
button("Home", "👁 Toggle ESP", Theme.panel2, function()
    Config.data.esp.enabled = not Config.data.esp.enabled; Config.save(); updateStatus()
end)
button("Home", "🌐 Fullbright", Theme.panel2, function()
    local f = Config.data.visuals.fullbright
    Config.data.visuals.fullbright = not f
    Lighting.Brightness = f and 1 or 2
    Lighting.Ambient = f and Color3.fromRGB(70,70,70) or Color3.fromRGB(200,200,200)
    Config.save()
end)
button("Home", "💾 Salvar Config", Theme.ok, function() Config.save() end)
button("Home", "⚠ Resetar Tudo", Theme.bad, function() Config.reset(); updateStatus() end)

local ac = Config.data.aimbot
section("Aimbot", "PRINCIPAIS")
toggle("Aimbot", "Enable Aimbot", ac.enabled, function(v) ac.enabled = v; updateStatus(); if _G.updateStateBubble then _G.updateStateBubble() end end)
keybind("Aimbot", "Tecla de Ativação", ac.key, function(k) ac.key = k end)
slider("Aimbot", "FOV", 20, 600, ac.fov, function(v) ac.fov = v end)
slider("Aimbot", "Smoothness", 1, 100, math.floor(ac.smooth*100), function(v) ac.smooth = v/100 end)
slider("Aimbot", "Aim Distance", 50, 2000, ac.distance, function(v) ac.distance = v end)
dropdown("Aimbot", "Região Corporal", {"Head","Neck","Chest","Torso"}, ac.part, function(v) ac.part = v end)
dropdown("Aimbot", "Prioridade", {"Distance","Center","Health"}, ac.priority, function(v) ac.priority = v end)
toggle("Aimbot", "Target Lock", ac.lock, function(v) ac.lock = v end)
section("Aimbot", "PREDIÇÃO")
toggle("Aimbot", "Ativar Aim Prediction", ac.predictionEnabled, function(v) ac.predictionEnabled = v end)
slider("Aimbot", "Prediction", 0, 50, math.floor(ac.prediction*100), function(v) ac.prediction = v/100 end)
section("Aimbot", "FILTROS")
local fc = Config.data.filters
toggle("Aimbot", "Team Check", fc.teamCheck, function(v) fc.teamCheck = v end)
toggle("Aimbot", "Ignorar Mortos", fc.ignoreDead, function(v) fc.ignoreDead = v end)
toggle("Aimbot", "Ignorar Invisíveis", fc.ignoreInvisible, function(v) fc.ignoreInvisible = v end)
toggle("Aimbot", "Ignorar NPCs", fc.ignoreNpcs, function(v) fc.ignoreNpcs = v end)
toggle("Aimbot", "Ignorar Sem Equipe", fc.ignoreNoTeam, function(v) fc.ignoreNoTeam = v end)
toggle("Aimbot", "Apenas Pouca Vida", fc.onlyLowHealth, function(v) fc.onlyLowHealth = v end)
slider("Aimbot", "Limite Vida Baixa", 1, 100, fc.lowHealthThreshold, function(v) fc.lowHealthThreshold = v end)

local ec = Config.data.esp
section("ESP", "VISUALIZAÇÃO")
toggle("ESP", "Enable ESP", ec.enabled, function(v) ec.enabled = v; updateStatus() end)
toggle("ESP", "Box ESP", ec.box, function(v) ec.box = v end)
toggle("ESP", "Corner Box", ec.cornerBox, function(v) ec.cornerBox = v end)
toggle("ESP", "Skeleton ESP", ec.skeleton, function(v) ec.skeleton = v end)
toggle("ESP", "Head ESP", ec.head, function(v) ec.head = v end)
toggle("ESP", "Name ESP", ec.name, function(v) ec.name = v end)
toggle("ESP", "Distance ESP", ec.distance, function(v) ec.distance = v end)
toggle("ESP", "Health ESP", ec.health, function(v) ec.health = v end)
toggle("ESP", "Health Bar", ec.healthBar, function(v) ec.healthBar = v end)
toggle("ESP", "Team ESP", ec.team, function(v) ec.team = v end)
toggle("ESP", "Line ESP", ec.line, function(v) ec.line = v end)
toggle("ESP", "Glow ESP", ec.glow, function(v) ec.glow = v end)
section("ESP", "PERSONALIZAÇÃO")
slider("ESP", "Transparência", 0, 100, math.floor(ec.transparency*100), function(v) ec.transparency = v/100 end)
slider("ESP", "Tamanho Texto", 8, 24, ec.textSize, function(v) ec.textSize = v end)
slider("ESP", "Distância Máx Render", 50, 3000, ec.maxDistance, function(v) ec.maxDistance = v end)
slider("ESP", "Largura Health Bar", 1, 10, ec.healthBarWidth, function(v) ec.healthBarWidth = v end)
toggle("ESP", "Mostrar Inimigos", ec.showEnemies, function(v) ec.showEnemies = v end)
toggle("ESP", "Mostrar Aliados", ec.showAllies, function(v) ec.showAllies = v end)

local tc = Config.data.team
section("Team", "CONFIGURAÇÃO")
toggle("Team", "Team Check ON/OFF", tc.enabled, function(v) tc.enabled = v end)
toggle("Team", "Mostrar Nome Equipe", tc.showTeamName, function(v) tc.showTeamName = v end)
toggle("Team", "Destacar Inimigos", tc.highlightEnemies, function(v) tc.highlightEnemies = v end)
button("Team", "🔄 Atualizar Cores de Equipe", Theme.panel2, function()
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= lp then
            local tn = p.Team and p.Team.Name or "Sem Equipe"
            if not tc.teamColors[tn] then
                tc.teamColors[tn] = Color3.fromHSV((p.UserId % 360) / 360, 0.7, 1)
            end
        end
    end
    Config.save()
end)
button("Team", "🔄 Limpar Cores", Theme.bad, function() tc.teamColors = {}; Config.save() end)

local playersFrame = Instance.new("Frame")
playersFrame.Size = UDim2.new(1, 0, 0, 0); playersFrame.AutomaticSize = Enum.AutomaticSize.Y
playersFrame.BackgroundTransparency = 1; playersFrame.Parent = UI.frames["Players"].scroll
local pll = Instance.new("UIListLayout"); pll.Padding = UDim.new(0, 4); pll.Parent = playersFrame

local function renderPlayers()
    for _, c in ipairs(playersFrame:GetChildren()) do
        if c:IsA("Frame") or c:IsA("TextButton") then c:Destroy() end
    end
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= lp then
            local row = Instance.new("Frame")
            row.Size = UDim2.new(1, 0, 0, 30); row.BackgroundColor3 = Theme.panel
            row.Parent = playersFrame; corner(row, 5)
            local tn = p.Team and p.Team.Name or "Sem Equipe"
            local color = tc.teamColors[tn] or Theme.accent
            local dot = Instance.new("Frame")
            dot.Size = UDim2.new(0, 8, 0, 8); dot.Position = UDim2.new(0, 10, 0.5, -4)
            dot.BackgroundColor3 = color; dot.Parent = row; corner(dot, 4)
            local dist = "?"
            if p.Character and p.Character:FindFirstChild("HumanoidRootPart") and lp.Character and lp.Character:FindFirstChild("HumanoidRootPart") then
                dist = math.floor((p.Character.HumanoidRootPart.Position - lp.Character.HumanoidRootPart.Position).Magnitude) .. "m"
            end
            local hum = p.Character and p.Character:FindFirstChildOfClass("Humanoid")
            local hp = hum and math.floor(hum.Health) or "?"
            local txt = label(row, string.format("%s • %s • %s • ❤%s", p.Name, tn, dist, hp), 11, Theme.text)
            txt.Size = UDim2.new(1, -30, 1, 0); txt.Position = UDim2.new(0, 24, 0, 0)
        end
    end
end
button("Players", "🔄 Atualizar Lista", Theme.panel2, renderPlayers)
renderPlayers()
task.spawn(function()
    while true do task.wait(3); if UI.activeTab == "Players" then renderPlayers() end end
end)

section("Visuals", "FOV CIRCLE")
toggle("Visuals", "Mostrar Círculo FOV", ac.showFov, function(v) ac.showFov = v end)
slider("Visuals", "Espessura FOV", 1, 10, ac.fovThickness, function(v) ac.fovThickness = v end)
section("Visuals", "AMBIENTE")
toggle("Visuals", "Fullbright", Config.data.visuals.fullbright, function(v)
    Config.data.visuals.fullbright = v
    Lighting.Brightness = v and 2 or 1
    Lighting.Ambient = v and Color3.fromRGB(200,200,200) or Color3.fromRGB(70,70,70)
end)
button("Visuals", "Noclip ON/OFF", Theme.panel2, function()
    if lp.Character then
        for _, p in ipairs(lp.Character:GetDescendants()) do
            if p:IsA("BasePart") then p.CanCollide = false end
        end
    end
end)

section("Settings", "INTERFACE")
slider("Settings", "Escala UI", 70, 150, math.floor(Config.data.ui.scale*100), function(v)
    Config.data.ui.scale = v/100
    main.Size = UDim2.new(0, 720*(v/100), 0, 480*(v/100))
end)
slider("Settings", "Transparência Menu", 0, 70, math.floor(Config.data.ui.transparency*100), function(v)
    Config.data.ui.transparency = v/100; main.BackgroundTransparency = v/100
end)
button("Settings", "💾 Salvar", Theme.ok, function() Config.save() end)
button("Settings", "⚠ Restaurar Padrões", Theme.bad, function() Config.reset(); updateStatus() end)

section("Profiles", "PERFIS")
local profileInput = Instance.new("TextBox")
profileInput.Size = UDim2.new(1, 0, 0, 30); profileInput.BackgroundColor3 = Theme.panel
profileInput.PlaceholderText = "Nome do perfil..."; profileInput.Text = ""
profileInput.TextColor3 = Theme.text; profileInput.PlaceholderColor3 = Theme.textDim
profileInput.Font = Theme.font; profileInput.TextSize = 12
profileInput.Parent = UI.frames["Profiles"].scroll; corner(profileInput, 6); pad(profileInput, 0, 8, 0, 0)
button("Profiles", "💾 Salvar Perfil", Theme.ok, function()
    local name = profileInput.Text
    if name and name ~= "" then
        Config.data.profiles.saved[name] = Config.export()
        Config.data.profiles.active = name; Config.save()
    end
end)
button("Profiles", "📤 Exportar (clipboard)", Theme.panel2, function()
    if setclipboard then setclipboard(Config.export()); print("[" .. SCRIPT_NAME .. "] copiado") end
end)
button("Profiles", "📥 Importar (clipboard)", Theme.panel2, function()
    local str = getclipboard and getclipboard()
    if str and Config.import(str) then print("[" .. SCRIPT_NAME .. "] importado"); updateStatus() end
end)
button("Profiles", "⚠ Resetar Tudo", Theme.bad, function() Config.reset(); updateStatus() end)

-- ============================================================
-- ABA ABOUT COM EFEITO RGB
-- ============================================================
local aboutScroll = UI.frames["About"].scroll

local aboutTitle = Instance.new("TextLabel")
aboutTitle.Size = UDim2.new(1, 0, 0, 50)
aboutTitle.BackgroundTransparency = 1
aboutTitle.Text = "DANNY SCRIPT"
aboutTitle.TextColor3 = Color3.fromRGB(0, 200, 255)
aboutTitle.Font = Enum.Font.GothamBlack
aboutTitle.TextSize = 28
aboutTitle.TextXAlignment = Enum.TextXAlignment.Center
aboutTitle.Parent = aboutScroll

local titleStroke = Instance.new("UIStroke")
titleStroke.Thickness = 2
titleStroke.Color = Color3.fromRGB(255, 0, 255)
titleStroke.Parent = aboutTitle

local aboutSub = Instance.new("TextLabel")
aboutSub.Size = UDim2.new(1, 0, 0, 24)
aboutSub.BackgroundTransparency = 1
aboutSub.Text = "by Danny  •  v" .. SCRIPT_VERSION
aboutSub.TextColor3 = Color3.fromRGB(180, 180, 200)
aboutSub.Font = Enum.Font.GothamMedium
aboutSub.TextSize = 13
aboutSub.TextXAlignment = Enum.TextXAlignment.Center
aboutSub.Parent = aboutScroll

local divider = Instance.new("Frame")
divider.Size = UDim2.new(1, 0, 0, 2)
divider.BackgroundColor3 = Color3.fromRGB(255, 0, 255)
divider.BorderSizePixel = 0
divider.Parent = aboutScroll

local followLabel = Instance.new("TextLabel")
followLabel.Size = UDim2.new(1, 0, 0, 28)
followLabel.BackgroundTransparency = 1
followLabel.Text = "✦ SIGA-ME NAS REDES ✦"
followLabel.TextColor3 = Color3.fromRGB(255, 200, 60)
followLabel.Font = Enum.Font.GothamBold
followLabel.TextSize = 14
followLabel.TextXAlignment = Enum.TextXAlignment.Center
followLabel.Parent = aboutScroll

local ytBtn = Instance.new("TextButton")
ytBtn.Size = UDim2.new(1, 0, 0, 44)
ytBtn.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
ytBtn.Text = ""
ytBtn.AutoButtonColor = false
ytBtn.BorderSizePixel = 0
ytBtn.Parent = aboutScroll
corner(ytBtn, 8)

local ytGradient = Instance.new("UIGradient")
ytGradient.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 0, 0)),
    ColorSequenceKeypoint.new(0.5, Color3.fromRGB(255, 80, 80)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(200, 0, 0)),
})
ytGradient.Parent = ytBtn

local ytIcon = Instance.new("TextLabel")
ytIcon.Size = UDim2.new(0, 40, 1, 0)
ytIcon.Position = UDim2.new(0, 8, 0, 0)
ytIcon.BackgroundTransparency = 1
ytIcon.Text = "▶"
ytIcon.TextColor3 = Color3.new(1,1,1)
ytIcon.Font = Enum.Font.GothamBlack
ytIcon.TextSize = 24
ytIcon.Parent = ytBtn

local ytText = Instance.new("TextLabel")
ytText.Size = UDim2.new(1, -60, 1, 0)
ytText.Position = UDim2.new(0, 50, 0, 0)
ytText.BackgroundTransparency = 1
ytText.Text = "YouTube  •  @dehmencia"
ytText.TextColor3 = Color3.new(1,1,1)
ytText.Font = Enum.Font.GothamBold
ytText.TextSize = 15
ytText.TextXAlignment = Enum.TextXAlignment.Left
ytText.Parent = ytBtn

local ytStroke = Instance.new("UIStroke")
ytStroke.Thickness = 1.5
ytStroke.Color = Color3.fromRGB(255, 255, 255)
ytStroke.Transparency = 0.3
ytStroke.Parent = ytBtn

local ttBtn = Instance.new("TextButton")
ttBtn.Size = UDim2.new(1, 0, 0, 44)
ttBtn.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
ttBtn.Text = ""
ttBtn.AutoButtonColor = false
ttBtn.BorderSizePixel = 0
ttBtn.Parent = aboutScroll
corner(ttBtn, 8)

local ttGradient = Instance.new("UIGradient")
ttGradient.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(0, 0, 0)),
    ColorSequenceKeypoint.new(0.4, Color3.fromRGB(0, 200, 255)),
    ColorSequenceKeypoint.new(0.7, Color3.fromRGB(255, 0, 100)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(0, 0, 0)),
})
ttGradient.Parent = ttBtn

local ttIcon = Instance.new("TextLabel")
ttIcon.Size = UDim2.new(0, 40, 1, 0)
ttIcon.Position = UDim2.new(0, 8, 0, 0)
ttIcon.BackgroundTransparency = 1
ttIcon.Text = "♪"
ttIcon.TextColor3 = Color3.new(1,1,1)
ttIcon.Font = Enum.Font.GothamBlack
ttIcon.TextSize = 24
ttIcon.Parent = ttBtn

local ttText = Instance.new("TextLabel")
ttText.Size = UDim2.new(1, -60, 1, 0)
ttText.Position = UDim2.new(0, 50, 0, 0)
ttText.BackgroundTransparency = 1
ttText.Text = "TikTok  •  @dehmencia"
ttText.TextColor3 = Color3.new(1,1,1)
ttText.Font = Enum.Font.GothamBold
ttText.TextSize = 15
ttText.TextXAlignment = Enum.TextXAlignment.Left
ttText.Parent = ttBtn

local ttStroke = Instance.new("UIStroke")
ttStroke.Thickness = 1.5
ttStroke.Color = Color3.fromRGB(255, 255, 255)
ttStroke.Transparency = 0.3
ttStroke.Parent = ttBtn

local hint = Instance.new("TextLabel")
hint.Size = UDim2.new(1, 0, 0, 20)
hint.BackgroundTransparency = 1
hint.Text = "(clique para copiar o link e abrir no navegador)"
hint.TextColor3 = Color3.fromRGB(140, 140, 160)
hint.Font = Enum.Font.Gotham
hint.TextSize = 10
hint.TextXAlignment = Enum.TextXAlignment.Center
hint.Parent = aboutScroll

local divider2 = Instance.new("Frame")
divider2.Size = UDim2.new(1, 0, 0, 2)
divider2.BackgroundColor3 = Color3.fromRGB(120, 80, 255)
divider2.BorderSizePixel = 0
divider2.Parent = aboutScroll

local aboutInfo = label(aboutScroll, "DANNY SCRIPT v" .. SCRIPT_VERSION .. "  •  Autor: " .. AUTHOR .. "\n\nAtalhos:\n• RightShift — abrir/fechar menu\n• Tecla configurada — toggle aimbot\n• Bolha (foto) — clique abre menu, duplo clique aimbot\n• Pílula DANNY AIM — clique alterna aimbot\n\nFuncionalidades:\n• Aimbot com FOV, smooth, prediction\n• ESP: box, corner, skeleton, health bar\n• Team check com cores dinâmicas\n• Player list ao vivo\n• Perfis salvos em JSON\n• Duas bolhas flutuantes independentes", 12, Theme.text)
aboutInfo.Size = UDim2.new(1, 0, 0, 320)
aboutInfo.TextWrapped = true
aboutInfo.TextYAlignment = Enum.TextYAlignment.Top

local function copyAndNotify(link, name)
    if setclipboard then
        setclipboard(link)
    end
    hint.Text = "✓ Link do " .. name .. " copiado! Cole no navegador."
    hint.TextColor3 = Color3.fromRGB(80, 220, 120)
    task.delay(3, function()
        hint.Text = "(clique para copiar o link e abrir no navegador)"
        hint.TextColor3 = Color3.fromRGB(140, 140, 160)
    end)
end

ytBtn.MouseButton1Click:Connect(function()
    copyAndNotify(SOCIAL.youtube, "YouTube")
end)
ttBtn.MouseButton1Click:Connect(function()
    copyAndNotify(SOCIAL.tiktok, "TikTok")
end)

-- ============================================================
-- ANIMAÇÃO RGB
-- ============================================================
task.spawn(function()
    local hue = 0
    while true do
        task.wait(0.05)
        hue = (hue + 0.01) % 1
        local c1 = Color3.fromHSV(hue, 0.9, 1)
        local c2 = Color3.fromHSV((hue + 0.33) % 1, 0.9, 1)
        local c3 = Color3.fromHSV((hue + 0.66) % 1, 0.9, 1)

        aboutTitle.TextColor3 = c1
        titleStroke.Color = c2
        divider.BackgroundColor3 = c3

        ytGradient.Color = ColorSequence.new({
            ColorSequenceKeypoint.new(0, c1),
            ColorSequenceKeypoint.new(0.5, c2),
            ColorSequenceKeypoint.new(1, c1),
        })
        ttGradient.Color = ColorSequence.new({
            ColorSequenceKeypoint.new(0, c2),
            ColorSequenceKeypoint.new(0.5, c3),
            ColorSequenceKeypoint.new(1, c2),
        })
        followLabel.TextColor3 = c3
        divider2.BackgroundColor3 = c1

        local pulse = 0.3 + math.abs(math.sin(tick() * 2)) * 0.4
        ytStroke.Transparency = pulse
        ttStroke.Transparency = pulse
        titleStroke.Thickness = 1.5 + math.abs(math.sin(tick() * 3)) * 1.5
    end
end)

-- ============================================================
-- BUSCA
-- ============================================================
search:GetPropertyChangedSignal("Text"):Connect(function()
    local q = search.Text:lower()
    for _, frame in pairs(UI.frames) do
        for _, w in ipairs(frame.scroll:GetChildren()) do
            if (w:IsA("TextButton") or w:IsA("Frame")) and w.Name ~= "UIListLayout" and w.Name ~= "UIPadding" then
                if q == "" then w.Visible = true
                else w.Visible = w.Name:lower():find(q) ~= nil end
            end
        end
    end
end)

-- ============================================================
-- TARGETING
-- ============================================================
local Targeting = {}
function Targeting.isAlive(p)
    if not p.Character then return false end
    local hum = p.Character:FindFirstChildOfClass("Humanoid")
    return hum and hum.Health > 0
end
function Targeting.getRoot(p) return p.Character and p.Character:FindFirstChild("HumanoidRootPart") end
function Targeting.isSameTeam(p)
    if not Config.data.team.enabled then return false end
    if not lp.Team or not p.Team then return false end
    return lp.Team == p.Team
end
function Targeting.getTarget()
    local best, bestScore = nil, math.huge
    local center = Vector2.new(cam.ViewportSize.X/2, cam.ViewportSize.Y/2)
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= lp then
            local skip = false
            if Config.data.filters.ignoreDead and not Targeting.isAlive(p) then skip = true end
            if Config.data.filters.teamCheck and Targeting.isSameTeam(p) then skip = true end
            if Config.data.filters.ignoreNoTeam and not p.Team then skip = true end
            if Config.data.filters.onlyLowHealth then
                local hum = p.Character and p.Character:FindFirstChildOfClass("Humanoid")
                if not hum or hum.Health > Config.data.filters.lowHealthThreshold then skip = true end
            end
            if not skip then
                local root = Targeting.getRoot(p)
                if root then
                    local dist = (root.Position - cam.CFrame.Position).Magnitude
                    if dist <= Config.data.aimbot.distance then
                        local part = p.Character:FindFirstChild(Config.data.aimbot.part) or p.Character:FindFirstChild("Head")
                        if part then
                            local sp, onScreen = cam:WorldToViewportPoint(part.Position)
                            if onScreen then
                                local sd = (Vector2.new(sp.X, sp.Y) - center).Magnitude
                                if sd <= Config.data.aimbot.fov then
                                    local score = Config.data.aimbot.priority == "Distance" and dist
                                        or Config.data.aimbot.priority == "Center" and sd
                                        or (1000 - (p.Character:FindFirstChildOfClass("Humanoid").Health))
                                    if score < bestScore then bestScore = score; best = p end
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

-- ============================================================
-- LOOPS
-- ============================================================
RunService.RenderStepped:Connect(function()
    if not Config.data.aimbot.enabled then return end
    local target = Targeting.getTarget()
    if target and target.Character then
        local part = target.Character:FindFirstChild(Config.data.aimbot.part) or target.Character:FindFirstChild("Head")
        if part then
            local aimPos = part.Position
            if Config.data.aimbot.predictionEnabled and part:IsA("BasePart") then
                aimPos = aimPos + part.Velocity * Config.data.aimbot.prediction
            end
            local look = CFrame.new(cam.CFrame.Position, aimPos)
            cam.CFrame = cam.CFrame:Lerp(look, Config.data.aimbot.smooth)
        end
    end
end)

local fovCircle = Instance.new("Frame")
fovCircle.Size = UDim2.new(0, 300, 0, 300)
fovCircle.Position = UDim2.new(0.5, -150, 0.5, -150)
fovCircle.BackgroundTransparency = 1; fovCircle.BorderSizePixel = 0
fovCircle.Parent = gui; corner(fovCircle, 150)
stroke(fovCircle, ac.fovColor, 1).Transparency = 0.5

RunService.RenderStepped:Connect(function()
    if not Config.data.aimbot.showFov or not Config.data.aimbot.enabled then
        fovCircle.Visible = false
    else
        fovCircle.Visible = true
        local size = Config.data.aimbot.fov * 2
        fovCircle.Size = UDim2.new(0, size, 0, size)
        fovCircle.Position = UDim2.new(0.5, -size/2, 0.5, -size/2)
        local s = fovCircle:FindFirstChildOfClass("UIStroke")
        if s then s.Color = Config.data.aimbot.fovColor; s.Thickness = Config.data.aimbot.fovThickness end
    end
end)

local function applyESP(p)
    if p == lp or not p.Character then return end
    local char = p.Character
    local root = char:FindFirstChild("HumanoidRootPart")
    local head = char:FindFirstChild("Head")
    if not root or not head then return end
    local dist = (root.Position - cam.CFrame.Position).Magnitude
    if dist > Config.data.esp.maxDistance then return end
    local sameTeam = Targeting.isSameTeam(p)
    if sameTeam and not Config.data.esp.showAllies then return end
    if not sameTeam and not Config.data.esp.showEnemies then return end
    local color = sameTeam and Config.data.esp.allyColor or Config.data.esp.enemyColor

    if Config.data.esp.glow then
        local hl = char:FindFirstChild("XenoESP_HL")
        if not hl then
            hl = Instance.new("Highlight"); hl.Name = "XenoESP_HL"
            hl.FillTransparency = 0.65; hl.Parent = char
        end
        hl.Enabled = Config.data.esp.enabled; hl.FillColor = color; hl.OutlineColor = color
    else
        local hl = char:FindFirstChild("XenoESP_HL"); if hl then hl:Destroy() end
    end

    local bb = head:FindFirstChild("XenoESP_BB")
    if not bb then
        bb = Instance.new("BillboardGui")
        bb.Name = "XenoESP_BB"; bb.Size = UDim2.new(0, 200, 0, 60)
        bb.StudsOffset = Vector3.new(0, 2.5, 0); bb.AlwaysOnTop = true; bb.Parent = head
        local txt = Instance.new("TextLabel")
        txt.Name = "Info"; txt.Size = UDim2.new(1, 0, 1, 0); txt.BackgroundTransparency = 1
        txt.TextColor3 = color; txt.Font = Theme.fontBold; txt.TextSize = Config.data.esp.textSize
        txt.TextStrokeTransparency = 0.5; txt.Parent = bb
    end
    bb.Enabled = Config.data.esp.enabled and (Config.data.esp.name or Config.data.esp.distance or Config.data.esp.health or Config.data.esp.team)
    if bb.Enabled then
        local parts = {}
        if Config.data.esp.name then table.insert(parts, p.Name) end
        if Config.data.esp.team and Config.data.team.showTeamName then
            table.insert(parts, p.Team and p.Team.Name or "Sem Equipe")
        end
        if Config.data.esp.distance then table.insert(parts, math.floor(dist) .. "m") end
        if Config.data.esp.health then
            local hum = char:FindFirstChildOfClass("Humanoid")
            if hum then table.insert(parts, "❤" .. math.floor(hum.Health)) end
        end
        local txt = bb:FindFirstChild("Info")
        if txt then txt.Text = table.concat(parts, " | "); txt.TextColor3 = color; txt.TextSize = Config.data.esp.textSize end
    end
end

RunService.Heartbeat:Connect(function()
    if not Config.data.esp.enabled then
        for _, p in ipairs(Players:GetPlayers()) do
            if p.Character then
                local h = p.Character:FindFirstChild("XenoESP_HL"); if h then h:Destroy() end
                local head = p.Character:FindFirstChild("Head")
                if head then
                    local b = head:FindFirstChild("XenoESP_BB"); if b then b:Destroy() end
                end
            end
        end
        return
    end
    for _, p in ipairs(Players:GetPlayers()) do applyESP(p) end
end)

RunService.Stepped:Connect(function()
    if Config.data.visuals.noclip and lp.Character then
        for _, p in ipairs(lp.Character:GetDescendants()) do
            if p:IsA("BasePart") then p.CanCollide = false end
        end
    end
end)

-- ============================================================
-- BOLHA DO MENU (FOTO + ARO RGB + MIRA + LABEL)
-- ============================================================
local bubbleWrapper = Instance.new("Frame")
bubbleWrapper.Name = "BubbleWrapper"
bubbleWrapper.Size = UDim2.new(0, 74, 0, 96)
bubbleWrapper.Position = Config.data.ui.bubblePos
bubbleWrapper.BackgroundTransparency = 1
bubbleWrapper.ZIndex = 10
bubbleWrapper.Parent = gui

local bubble = Instance.new("ImageButton")
bubble.Name = "XenoBubble"
bubble.Size = UDim2.new(0, 64, 0, 64)
bubble.Position = UDim2.new(0, 5, 0, 0)
bubble.BackgroundColor3 = Theme.panel2
bubble.BackgroundTransparency = 0
bubble.Image = BUBBLE_IMAGE
bubble.ImageColor3 = Color3.new(1,1,1)
bubble.ScaleType = Enum.ScaleType.Crop
bubble.AutoButtonColor = false
bubble.BorderSizePixel = 0
bubble.ZIndex = 11
bubble.Parent = bubbleWrapper
corner(bubble, 32)

local ring = Instance.new("Frame")
ring.Size = UDim2.new(1, 6, 1, 6)
ring.Position = UDim2.new(0, -3, 0, -3)
ring.BackgroundTransparency = 1
ring.ZIndex = 12
ring.Parent = bubble
corner(ring, 35)
local ringStroke = Instance.new("UIStroke")
ringStroke.Thickness = 3
ringStroke.Color = Theme.accent
ringStroke.Transparency = 0.1
ringStroke.Parent = ring

local glow = Instance.new("Frame")
glow.Size = UDim2.new(1, 0, 1, 0)
glow.BackgroundColor3 = Theme.accent
glow.BackgroundTransparency = 0.85
glow.BorderSizePixel = 0
glow.ZIndex = 13
glow.Parent = bubble
corner(glow, 32)

local aimIcon = Instance.new("TextLabel")
aimIcon.Size = UDim2.new(0, 22, 0, 22)
aimIcon.Position = UDim2.new(1, -26, 1, -26)
aimIcon.BackgroundColor3 = Theme.bg
aimIcon.Text = "◎"
aimIcon.TextColor3 = Theme.accent
aimIcon.Font = Enum.Font.GothamBlack
aimIcon.TextSize = 14
aimIcon.ZIndex = 14
aimIcon.Parent = bubble
corner(aimIcon, 11)
local aimIconStroke = Instance.new("UIStroke")
aimIconStroke.Color = Theme.accent
aimIconStroke.Thickness = 1.5
aimIconStroke.Parent = aimIcon

local stateLabel = Instance.new("TextLabel")
stateLabel.Size = UDim2.new(1, 0, 0, 20)
stateLabel.Position = UDim2.new(0, 0, 0, 68)
stateLabel.BackgroundColor3 = Theme.bg
stateLabel.BackgroundTransparency = 0.25
stateLabel.Text = "OFF"
stateLabel.TextColor3 = Theme.bad
stateLabel.Font = Theme.fontBold
stateLabel.TextSize = 10
stateLabel.ZIndex = 13
stateLabel.Parent = bubbleWrapper
corner(stateLabel, 10)
local stateLabelStroke = Instance.new("UIStroke")
stateLabelStroke.Color = Theme.bad
stateLabelStroke.Thickness = 1
stateLabelStroke.Parent = stateLabel

updateBubbleLook = function()
    local stateColor
    if main.Visible then
        stateColor = Theme.warn
    elseif Config.data.aimbot.enabled then
        stateColor = Theme.ok
    else
        stateColor = Theme.bad
    end

    ringStroke.Color = stateColor
    glow.BackgroundColor3 = stateColor
    aimIcon.TextColor3 = stateColor
    aimIconStroke.Color = stateColor
    stateLabel.TextColor3 = stateColor
    stateLabelStroke.Color = stateColor
    stateLabel.Text = main.Visible and "MENU" or (Config.data.aimbot.enabled and "ON" or "OFF")

    if Config.data.aimbot.enabled and not main.Visible then
        ringStroke.Thickness = 3 + math.abs(math.sin(tick() * 4)) * 2
        glow.BackgroundTransparency = 0.8 + math.abs(math.sin(tick() * 3)) * 0.1
    else
        ringStroke.Thickness = 3
        glow.BackgroundTransparency = 0.9
    end
end

task.spawn(function()
    local hue = 0
    while true do
        task.wait(0.05)
        if Config.data.aimbot.enabled and not main.Visible then
            hue = (hue + 0.02) % 1
            local rgb = Color3.fromHSV(hue, 0.85, 1)
            ringStroke.Color = rgb
            aimIcon.TextColor3 = rgb
            aimIconStroke.Color = rgb
        end
    end
end)

-- ============================================================
-- PÍLULA DE ESTADO
-- ============================================================
local stateBubble = Instance.new("TextButton")
stateBubble.Name = "AimStateBubble"; stateBubble.Size = UDim2.new(0, 150, 0, 34)
stateBubble.Position = Config.data.ui.stateBubblePos
stateBubble.BackgroundColor3 = Theme.bad; stateBubble.Text = "DANNY AIM: DESATIVADO"
stateBubble.TextColor3 = Color3.new(1,1,1); stateBubble.TextSize = 11
stateBubble.Font = Theme.fontBold; stateBubble.AutoButtonColor = false
stateBubble.BorderSizePixel = 0; stateBubble.ZIndex = 10; stateBubble.Parent = gui
corner(stateBubble, 17); stroke(stateBubble, Color3.new(1,1,1), 1.5).Transparency = 0.4

function _G.updateStateBubble()
    if Config.data.aimbot.enabled then
        stateBubble.BackgroundColor3 = Theme.ok
        stateBubble.Text = "DANNY AIM: ATIVADO"
    else
        stateBubble.BackgroundColor3 = Theme.bad
        stateBubble.Text = "DANNY AIM: DESATIVADO"
    end
    updateBubbleLook()
end

local sdragging, sdragStart, sstartPos = false, nil, nil
stateBubble.InputBegan:Connect(function(i)
    if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
        sdragging = true; sdragStart = i.Position; sstartPos = stateBubble.Position
    end
end)
UserInputService.InputChanged:Connect(function(i)
    if sdragging and (i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch) then
        local d = i.Position - sdragStart
        stateBubble.Position = UDim2.new(sstartPos.X.Scale, sstartPos.X.Offset + d.X, sstartPos.Y.Scale, sstartPos.Y.Offset + d.Y)
    end
end)
UserInputService.InputEnded:Connect(function(i)
    if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
        if sdragging then
            sdragging = false
            Config.data.ui.stateBubblePos = stateBubble.Position; Config.save()
        end
    end
end)
stateBubble.MouseButton1Click:Connect(function()
    Config.data.aimbot.enabled = not Config.data.aimbot.enabled
    Config.save(); updateStatus(); _G.updateStateBubble()
end)

-- ============================================================
-- BOLHA: ARRASTAR/CLICAR (wrapper)
-- ============================================================
local dragging, dragStart, startPos = false, nil, nil
local lastClick = 0

bubble.InputBegan:Connect(function(i)
    if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
        dragging = true; dragStart = i.Position; startPos = bubbleWrapper.Position
        TweenService:Create(bubble, TweenInfo.new(0.1), {Size = UDim2.new(0, 58, 0, 58), Position = UDim2.new(0, 8, 0, 3)}):Play()
    end
end)
UserInputService.InputChanged:Connect(function(i)
    if dragging and (i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch) then
        local d = i.Position - dragStart
        bubbleWrapper.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + d.X, startPos.Y.Scale, startPos.Y.Offset + d.Y)
    end
end)
UserInputService.InputEnded:Connect(function(i)
    if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
        if dragging then
            dragging = false
            TweenService:Create(bubble, TweenInfo.new(0.15), {Size = UDim2.new(0, 64, 0, 64), Position = UDim2.new(0, 5, 0, 0)}):Play()
            Config.data.ui.bubblePos = bubbleWrapper.Position; Config.save()
            local now = tick()
            if now - lastClick < 0.35 then
                Config.data.aimbot.enabled = not Config.data.aimbot.enabled
                lastClick = 0; updateStatus(); _G.updateStateBubble()
            else
                lastClick = now
                task.delay(0.35, function()
                    if tick() - lastClick >= 0.34 then
                        main.Visible = not main.Visible
                        updateBubbleLook()
                    end
                end)
            end
            updateBubbleLook()
        end
    end
end)

-- ============================================================
-- KEYBINDS
-- ============================================================
UserInputService.InputBegan:Connect(function(i, gp)
    if gp then return end
    if i.KeyCode == Enum.KeyCode.RightShift then
        main.Visible = not main.Visible
        updateBubbleLook()
    elseif i.KeyCode == Config.data.aimbot.key then
        Config.data.aimbot.enabled = not Config.data.aimbot.enabled
        Config.save(); updateStatus(); _G.updateStateBubble()
        updateBubbleLook()
    end
end)

-- ============================================================
-- INIT
-- ============================================================
updateStatus()
updateBubbleLook()
_G.updateStateBubble()
print(string.format("[%s] v%s por %s carregado com sucesso.", SCRIPT_NAME, SCRIPT_VERSION, AUTHOR))
