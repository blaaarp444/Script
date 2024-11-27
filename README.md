-- Orion
local OrionLib = loadstring(game:HttpGet(('https://raw.githubusercontent.com/shlexware/Orion/main/source')))()

-- Window Hub
local Window = OrionLib:MakeWindow({Name = "Uri Hub", HidePremium = false, SaveConfig = true, IntroEnabled = false})

-- Toggles
local Toggles = {
    Train = false,
    Spawn = false,
    Inferno = false,
    AutoFarm = false,  -- Novo toggle para o ciclo de farm
}

-- Funções principais
local function Train()
    while Toggles.Train do
        local equipment = {
            "Weights", "BenchPresses", "Deadlift Weights", "Punching Bags",
            "Treadmills", "Chestpresses", "Bikes", "Weighted Vests",
            "Prowler Sleds", "Sit Up", "Jogging", "Push Up"
        }

        for _, tool in ipairs(equipment) do
            game:GetService("ReplicatedStorage").Aero.AeroRemoteServices.EquipmentHandler.Use:FireServer(tool)
        end

        game:GetService("ReplicatedStorage").Aero.AeroRemoteServices.QuestService.Claim:FireServer("GainStrength")
        task.wait(0.1)
    end
end

local function MoveToLocation(location)
    game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(unpack(location))
    task.wait(0.01)
end

-- Função para equipar a ferramenta "Smash"
local function EquipSmash()
    local smashTool = game.Players.LocalPlayer.Backpack:FindFirstChild("Smash")
    if smashTool then
        smashTool.Parent = game.Players.LocalPlayer.Character
        game.Players.LocalPlayer.Character:WaitForChild("Smash"):Activate()  -- Ativa a ferramenta automaticamente
    else
        warn("Ferramenta Smash não encontrada no inventário.")
    end
end

-- Função para atacar o boss com base no NPC
local function AttackBoss(boss)
    local npc = boss:FindFirstChild("Npc")
    if npc then
        -- Equipar e ativar a ferramenta Smash automaticamente
        EquipSmash()

        -- Espera até que o NPC seja derrotado (verifica o hp do NPC)
        local humanoid = npc:FindFirstChild("Humanoid")
        if humanoid then
            while humanoid and humanoid.Health > 0 do
                wait(1)  -- Atraso entre ataques, ajustável
            end
        end
    end
end

local function Spawn()
    while Toggles.Spawn do
        game:GetService("ReplicatedStorage").Aero.AeroRemoteServices.ClientHandler.UsePortal:FireServer("Starter Map")
        MoveToLocation({168.256, 544.023, -4617.658})
    end
end

local function Inferno()
    while Toggles.Inferno do
        game:GetService("ReplicatedStorage").Aero.AeroRemoteServices.ClientHandler.UsePortal:FireServer("Underworld")
        MoveToLocation({24347.130859375, 987.9713745117188, -4205.2998046875})  -- Coordenada corrigida do Inferno
    end
end

-- Função para monitorar e farmar os bosses com base no estado do NPC
local function MonitorAndFarmBosses()
    while Toggles.AutoFarm do
        -- Verifica o NPC do Inferno e começa a atacar quando ele estiver presente
        local infernoBoss = workspace.Maps.Underworld.Boss.Npc
        if infernoBoss and infernoBoss.Parent then
            AttackBoss(infernoBoss)  -- Ataca o boss do Inferno
        end

        -- Espera o primeiro boss morrer para começar a farmar o segundo
        while infernoBoss and infernoBoss.Parent and infernoBoss:FindFirstChild("Humanoid") and infernoBoss.Humanoid.Health > 0 do
            wait(1)  -- Atraso para continuar verificando
        end

        -- Quando o boss do Inferno morrer, teleporta para o Spawn
        game:GetService("ReplicatedStorage").Aero.AeroRemoteServices.ClientHandler.UsePortal:FireServer("Starter Map")
        MoveToLocation({168.256, 544.023, -4617.658})  -- Coordenada do inimigo Spawn

        -- Verifica o NPC do Spawn e começa a atacar quando ele estiver presente
        local spawnBoss = workspace.Maps["Starter Map"].Boss.Npc
        if spawnBoss and spawnBoss.Parent then
            AttackBoss(spawnBoss)  -- Ataca o boss do Spawn
        end

        -- Espera o boss do Spawn morrer antes de reiniciar o ciclo
        while spawnBoss and spawnBoss.Parent and spawnBoss:FindFirstChild("Humanoid") and spawnBoss.Humanoid.Health > 0 do
            wait(1)  -- Atraso para continuar verificando
        end

        -- Reinicia o ciclo de farm
        wait(1)
    end
end

-- Função de auto farm entre os dois mundos (Inferno -> Spawn)
local function AutoFarmCycle()
    while Toggles.AutoFarm do
        -- Teleporta para o mundo Inferno
        game:GetService("ReplicatedStorage").Aero.AeroRemoteServices.ClientHandler.UsePortal:FireServer("Underworld")
        MoveToLocation({24347.130859375, 987.9713745117188, -4205.2998046875})  -- Coordenada corrigida do inimigo Inferno
        MonitorAndFarmBosses()  -- Monitora e ataca o boss Inferno
        wait(1)  -- Espera 1 segundo após derrotar o boss Inferno

        -- Teleporta para o mundo Spawn
        game:GetService("ReplicatedStorage").Aero.AeroRemoteServices.ClientHandler.UsePortal:FireServer("Starter Map")
        MoveToLocation({168.256, 544.023, -4617.658})  -- Coordenada do inimigo Spawn
        MonitorAndFarmBosses()  -- Monitora e ataca o boss Spawn
        wait(1)  -- Espera 1 segundo após derrotar o boss Spawn
    end
end

-- GUI
local FarmTab = Window:MakeTab({Name = "Farms", Icon = "rbxassetid://4483345998", PremiumOnly = false})
FarmTab:AddSection({Name = "Auto Farm"})

FarmTab:AddToggle({
    Name = "Treinar Automático",
    Default = false,
    Callback = function(Value)
        Toggles.Train = Value
        if Value then Train() end
    end    
})

FarmTab:AddToggle({
    Name = "Farm Spawn Boss",
    Default = false,
    Callback = function(Value)
        Toggles.Spawn = Value
        if Value then Spawn() end
    end    
})

FarmTab:AddToggle({
    Name = "Farm Inferno Boss",
    Default = false,
    Callback = function(Value)
        Toggles.Inferno = Value
        if Value then Inferno() end
    end    
})

FarmTab:AddToggle({
    Name = "Auto Farm Bosses",
    Default = false,
    Callback = function(Value)
        Toggles.AutoFarm = Value
        if Value then AutoFarmCycle() end
    end    
})

-- Inicializar
OrionLib:Init()
