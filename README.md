-- [ Script Steal An Egg - Redesigned UI com Orion Library ]

local fn

fn = function(arg)
    local genv = typeof(getgenv) == "function" and getgenv() or _G

    if type(genv.ChilliDebugPrint) == "function" then
        pcall(genv.ChilliDebugPrint, arg)
    end
end

task.spawn(pcall, function()
    loadstring(
        game:HttpGet(
            "https://raw.githubusercontent.com/tienkhanh1/spicy/refs/heads/main/DiscordLink"
        )
    )()
end)

-- Carregamento do motor gráfico Orion Library
local OrionLib = loadstring(game:HttpGet('https://raw.githubusercontent.com/shlexware/Orion/main/source'))()

local Window = OrionLib:MakeWindow({
    Name = "Steal An Egg Hub | Nova Interface",
    HidePremium = false,
    SaveConfig = false,
    IntroText = "Steal An Egg Hub"
})

-- Definição dos Separadores (Tabs)
local Tabs = {
    ["Dr Scramble Event"] = Window:MakeTab({Name = "Dr Scramble", Icon = "rbxassetid://4483345998"}),
    ["Auto Steal"] = Window:MakeTab({Name = "Auto Steal", Icon = "rbxassetid://4483345998"}),
    ["Auto Place Egg"] = Window:MakeTab({Name = "Quinta & Ovos", Icon = "rbxassetid://4483345998"}),
    ["Auto Treadmill"] = Window:MakeTab({Name = "Esteira", Icon = "rbxassetid://4483345998"}),
    ["Auto Hatch & Equip"] = Window:MakeTab({Name = "Choco & Equipar", Icon = "rbxassetid://4483345998"}),
    ["Auto Sell"] = Window:MakeTab({Name = "Vendas", Icon = "rbxassetid://4483345998"}),
    ["Auto Fuse Machine"] = Window:MakeTab({Name = "Fusão & Loja", Icon = "rbxassetid://4483345998"}),
    ["Auto Favorite"] = Window:MakeTab({Name = "Favoritos", Icon = "rbxassetid://4483345998"}),
    ["Auto Rift & Boss"] = Window:MakeTab({Name = "Rift & Boss", Icon = "rbxassetid://4483345998"})
}

-- Adaptador de compatibilidade de API
local v = {}
v.ManualQuickDefaults = {}
function v.Finalize() OrionLib:Init() end

function v:CreateWindow(cfg)
    local win = {}
    function win:GetDefaultTab() return win end
    
    function win:CreateSection(secCfg)
        local secName = secCfg.Name
        local tab = Tabs[secName] or Window:MakeTab({Name = secName, Icon = "rbxassetid://4483345998"})
        tab:AddSection({Name = secName})
        
        local sec = {}
        
        function sec:CreateToggle(opt)
            local toggleObj = {_val = opt.Default or false}
            tab:AddToggle({
                Name = opt.Name,
                Default = opt.Default or false,
                Callback = function(val)
                    toggleObj._val = val
                    if opt.Callback then opt.Callback(val) end
                end
            })
            function toggleObj:Get() return toggleObj._val end
            function toggleObj:GetValue() return toggleObj._val end
            function toggleObj:Set(val) toggleObj._val = val end
            return toggleObj
        end
        
        function sec:CreateDropdown(opt)
            local dropObj = {_val = opt.Default}
            tab:AddDropdown({
                Name = opt.Name,
                Default = opt.Default,
                Options = opt.Options or {},
                Callback = function(val)
                    dropObj._val = val
                    if opt.Callback then opt.Callback(val) end
                end
            })
            function dropObj:Get() return dropObj._val end
            return dropObj
        end
        
        function sec:CreateMultiDropdown(opt)
            local multiObj = {_val = opt.Default or {}}
            tab:AddDropdown({
                Name = opt.Name .. " (Seleção)",
                Default = type(opt.Default) == "table" and opt.Default[1] or opt.Default,
                Options = opt.Options or {},
                Callback = function(val)
                    if opt.Callback then opt.Callback({[val] = true}) end
                end
            })
            return multiObj
        end
        
        function sec:CreateSlider(opt)
            local sliderObj = {_val = opt.Default or opt.Min or 0}
            tab:AddSlider({
                Name = opt.Name,
                Min = opt.Min or 0,
                Max = opt.Max or 1000,
                Default = opt.Default or opt.Min or 0,
                Color = Color3.fromRGB(255, 85, 85),
                Increment = opt.Increment or 1,
                ValueName = type(opt.Unit) == "string" and opt.Unit or "",
                Callback = function(val)
                    sliderObj._val = val
                    if opt.Callback then opt.Callback(val) end
                end
            })
            function sliderObj:Get() return sliderObj._val end
            function sliderObj:Set(val) sliderObj._val = val end
            function sliderObj:SetRange(min, max) end
            return sliderObj
        end
        
        return sec
    end
    return win
end

v.ManualQuickDefaults = {
    PinnedFeatures = { "Player > Movement > Speed Boost", "Player > Movement > Boost Speed" },
    Keybinds = { ["Player > Movement > Speed Boost"] = "Q" },
    PinGroups = {},
    LeftCenterHidden = true,
}

local v2 = v:CreateWindow({ Name = "Chilli Hub - Steal An Egg", DefaultTab = "Farm" })
local defaultTab = v2:GetDefaultTab()

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local CoreGui = game:GetService("CoreGui")
local UserInputService = game:GetService("UserInputService")
local CollectionService = game:GetService("CollectionService")
game:GetService("LocalizationService")
local ProximityPromptService = game:GetService("ProximityPromptService")
local localPlayer = Players.LocalPlayer
local networking = ReplicatedStorage:WaitForChild("Packages"):WaitForChild("Networking")

local fn3 = function(arg)
    local ok, result = pcall(function()
        return require(arg())
    end)
    if not ok then result = ok end
    return result or nil
end

local tbl = {
    EggState = fn3(function() return ReplicatedStorage.Client.EggState end),
    AreaEggs = fn3(function() return ReplicatedStorage.Shared.Types.AreaEggs end),
    ToolGameplayGuard = fn3(function() return ReplicatedStorage.Client.ToolGameplayGuard end),
    Assets = fn3(function() return ReplicatedStorage.Data.Assets end),
    Guards = fn3(function() return ReplicatedStorage.Data.Guards end),
    EggRecords = fn3(function() return ReplicatedStorage.Shared.Util.EggRecords end),
    Mutations = fn3(function() return ReplicatedStorage.Shared.Modules.Mutations end),
    Save = fn3(function() return ReplicatedStorage.Shared.Save end),
    FuseKernel = fn3(function() return ReplicatedStorage.Shared.Util.FuseKernel end),
    AreaEggCycle = fn3(function() return ReplicatedStorage.Shared.Util.AreaEggCycle end),
    AreaEggResetWall = fn3(function() return ReplicatedStorage.Client.AreaEggResetWall end),
    AreaEggResetCycle = fn3(function() return ReplicatedStorage.Data.AreaEggResetCycle end),
    Gears = fn3(function() return ReplicatedStorage.Data.Gears end),
    Areas = fn3(function() return ReplicatedStorage.Data.Areas end),
    LimitedEgg = fn3(function() return ReplicatedStorage.Data.LimitedEgg end),
    BrainrotEgg = fn3(function() return ReplicatedStorage.Data.BrainrotEgg end),
    MonsterEgg = fn3(function() return ReplicatedStorage.Data.MonsterEgg end),
}

local function fn4()
    if typeof(gethui) == "function" then
        local ok, result = pcall(gethui)
        if ok and typeof(result) == "Instance" then
            return result
        end
    end
    return CoreGui
end

local v3 = fn4()
local fn5, fn6, fn7

do
    local v4 = Random.new()
    local str = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789"

    fn5 = function()
        local v5 = v4:NextInteger(12, 20)
        local v6 = table.create(v5)
        for i = 1, v5 do
            local v7 = v4:NextInteger(1, #str)
            v6[i] = string.sub(str, v7, v7)
        end
        return table.concat(v6)
    end
end

do
    local tbl2 = {}

    fn6 = function(arg)
        table.insert(tbl2, arg)
    end

    local str = "All"

    fn7 = function(arg)
        if type(arg) ~= "table" then return arg end
        local value = rawget(arg, "Instance")
        if typeof(value) ~= "Instance" then return arg end
        local flag = false

        local function fn8(arg2)
            if flag then return end
            if arg2.Text == "None" then
                flag = true
                arg2.Text = str
                flag = false
            end
        end

        local function fn9(arg2)
            if not arg2:IsA("TextLabel") or arg2.Name ~= "Value" then return end
            fn8(arg2)
            local connection = arg2:GetPropertyChangedSignal("Text"):Connect(function() fn8(arg2) end)
            fn6(function() pcall(function() connection:Disconnect() end) end)
        end

        for _, descendant in ipairs(value:GetDescendants()) do fn9(descendant) end
        local connection = value.DescendantAdded:Connect(fn9)
        fn6(function() pcall(function() connection:Disconnect() end) end)

        return arg
    end

    local genv = typeof(getgenv) == "function" and getgenv() or _G
    genv.ChilliHubSaeCleanup = function()
        for i = #tbl2, 1, -1 do pcall(tbl2[i]) end
        table.clear(tbl2)
    end
end

local tbl2
do
    local n, n2 = 0.35, 5
    local tbl3 = {}
    local flag = true

    tbl2 = {
        Add = function(arg)
            local tbl4 = { Run = arg, Gap = n, Idle = n2, Repeat = false, Hold = 0 }
            table.insert(tbl3, tbl4)
            return tbl4
        end,
        Wake = function() flag = true end,
        Backoff = function(arg, arg2)
            if arg then arg.Hold = tonumber(arg2) or 6 end
        end,
    }

    local connection = RunService.Heartbeat:Connect(function(arg)
        local v4 = flag
        flag = false
        for _, v5 in ipairs(tbl3) do
            v5.Gap = v5.Gap + arg
            v5.Idle = v5.Idle + arg
            if 0 < v5.Hold then
                v5.Hold = v5.Hold - arg
            elseif v5.Gap >= n and (v4 or v5.Repeat or v5.Idle >= n2) then
                v5.Gap = 0
                v5.Idle = 0
                local ok, result = pcall(v5.Run, v5)
                v5.Repeat = ok and result == true
            end
        end
    end)

    fn6(function() connection:Disconnect() end)
end

-- Inicialização das secções na nova interface gráfica por separadores
local v4 = defaultTab:CreateSection({ Name = "Dr Scramble Event", Expanded = false })
local v5 = defaultTab:CreateSection({ Name = "Auto Steal", Expanded = true })
local v6 = defaultTab:CreateSection({ Name = "Auto Place Egg", Expanded = false })
local v7 = defaultTab:CreateSection({ Name = "Auto Treadmill", Expanded = false })
local v8 = defaultTab:CreateSection({ Name = "Auto Hatch & Equip", Expanded = false })
local v9 = defaultTab:CreateSection({ Name = "Auto Sell", Expanded = false })
local v10 = defaultTab:CreateSection({ Name = "Auto Fuse Machine", Expanded = false })
local v11 = defaultTab:CreateSection({ Name = "Auto Favorite", Expanded = false })
local v12 = defaultTab:CreateSection({ Name = "Auto Rift & Boss", Expanded = false })

local tbl3 = { Paused = false }

local tbl4
local tbl5 = { "bat", "katana", "axe", "staff", "club", "hammer", "sword", "blade" }

tbl4 = {
    Steal = { Active = false, LastFinishedAt = 0, Carrying = false },
    Movement = { Owner = nil, PlaceWanted = false, StealFirst = false, MutationWanted = false, FracturedWanted = false },
    AntiGuard = { Enabled = false, Busy = false, BusySince = 0, HitArms = 0, Handle = nil, Render = nil },
    IsBatTool = function(arg)
        if typeof(arg) ~= "Instance" or not arg:IsA("Tool") then return false end
        if arg:GetAttribute("IsBat") == true then return true end
        local attribute = arg:GetAttribute("GearName")
        if type(attribute) == "string" then
            local gears = tbl.Gears
            local directory = type(gears) == "table" and gears.Directory or nil
            local flag = type(directory) == "table" and directory[attribute] or nil
            return type(flag) == "table" and flag.BatControllerData ~= nil
        end
        if arg:GetAttribute("ItemType") ~= nil then return false end
        local v13 = string.lower(arg.Name)
        for _, v14 in ipairs(tbl5) do
            if string.find(v13, v14, 1, true) then return true end
        end
        return false
    end,
}

tbl4.FindBat = function()
    local character = localPlayer.Character
    local tool = character and character:FindFirstChildWhichIsA("Tool")
    if tbl4.IsBatTool(tool) then return tool end
    local backpack = localPlayer:FindFirstChildOfClass("Backpack")
    if backpack then
        for _, child in ipairs(backpack:GetChildren()) do
            if tbl4.IsBatTool(child) then return child end
        end
    end
    return nil
end

tbl4.Root = function()
    local character = localPlayer.Character
    character = character and character:FindFirstChild("HumanoidRootPart")
    return (character and character:IsDescendantOf(workspace)) and character or nil
end

tbl4.Toggle = function(arg, arg2)
    if type(arg) ~= "table" then return arg2 == true end
    if type(arg.Get) == "function" then return arg:Get() == true end
    if type(arg.GetValue) == "function" then return arg:GetValue() == true end
    return arg2 == true
end

-- Configurações dos Controlos no Separador Auto Steal (v5)
local v13 = v5:CreateToggle({
    Name = "Auto Steal Ativo",
    Default = false,
    Callback = function(val)
        tbl4.Steal.Active = val
    end,
})

local tbl17 = {
    "Forest", "Desert", "Snow", "Lake", "Jungle", "Volcano",
    "Prehistoric", "Cosmic", "Abyss Ocean", "Cherry Blossom", "Light Dark", "Titan Temple"
}

v5:CreateMultiDropdown({
    Name = "Áreas Alvo",
    Options = tbl17,
    Default = tbl17,
    Callback = function(arg) end,
})

local tbl7 = { "Any", "Common", "Uncommon", "Rare", "Epic", "Legendary", "Mythic", "Cosmic", "Secret", "Divine" }
v5:CreateDropdown({
    Name = "Raridade Mínima",
    Options = tbl7,
    Default = tbl7[1],
    Callback = function(arg) end,
})

v5:CreateSlider({
    Name = "Velocidade de Tween",
    Min = 100,
    Max = 1000,
    Default = 400,
    Increment = 10,
    Unit = "studs/s",
    Callback = function(arg) end,
})

tbl4.AntiGuard.PanelHandle = v5:CreateToggle({
    Name = "Anti Guard V1",
    Default = false,
    Callback = function(arg)
        tbl4.AntiGuard.Enabled = arg
    end,
})

v.Finalize()
