--// Auto True Triple Katana Hunter Script
local HttpService = game:GetService("HttpService")
local TeleportService = game:GetService("TeleportService")
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer

local PLACE_ID = game.PlaceId
local visitedServers = {}

-- Lista de espadas necesarias
local requiredSwords = {
    "Saddi",
    "Shisui",
    "Wando"
}

-- Revisa si ya tienes todas las espadas necesarias
local function hasAllSwords()
    local backpack = player.Backpack
    local character = player.Character or player.CharacterAdded:Wait()

    for _, sword in ipairs(requiredSwords) do
        if not backpack:FindFirstChild(sword) and not character:FindFirstChild(sword) then
            return false
        end
    end
    return true
end

-- Intenta comprar la espada si el NPC está cerca
local function tryBuySword(npcName, swordName)
    local npc = workspace:FindFirstChild(npcName)
    if npc and npc:FindFirstChild("Head") then
        local root = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
        if root then
            root.CFrame = npc.Head.CFrame + Vector3.new(0, 3, 0)
            fireclickdetector(npc.Head:FindFirstChildOfClass("ClickDetector"))
            wait(0.5)
            -- Simula compra
            for _, gui in pairs(player.PlayerGui:GetDescendants()) do
                if gui:IsA("TextButton") and gui.Text:lower():find(swordName:lower()) then
                    gui:Activate()
                    break
                end
            end
        end
    end
end

-- Cambiar de servidor
local function hopServer()
    local servers = {}
    local cursor = ""
    local found = false

    while not found do
        local url = "https://games.roblox.com/v1/games/" .. PLACE_ID .. "/servers/Public?sortOrder=Asc&limit=100" .. (cursor ~= "" and "&cursor=" .. cursor or "")
        local response = game:HttpGet(url)
        local data = HttpService:JSONDecode(response)

        for _, server in pairs(data.data) do
            if server.playing < server.maxPlayers and not visitedServers[server.id] then
                visitedServers[server.id] = true
                TeleportService:TeleportToPlaceInstance(PLACE_ID, server.id, player)
                return
            end
        end

        if data.nextPageCursor then
            cursor = data.nextPageCursor
        else
            break
        end
    end
end

-- Revisión y ejecución principal
RunService.RenderStepped:Connect(function()
    if hasAllSwords() then
        print("Ya tienes las tres espadas legendarias.")
        return
    end

    local dealer = workspace:FindFirstChild("Legendary Sword Dealer")
    if dealer and dealer:FindFirstChild("Head") then
        -- Teletranspórtate al dealer y compra
        tryBuySword("Legendary Sword Dealer", "Saddi")
        tryBuySword("Legendary Sword Dealer", "Shisui")
        tryBuySword("Legendary Sword Dealer", "Wando")
    else
        -- Cambia de servidor
        hopServer()
    end
end)

-- Re-ejecuta el script después del teleport
queue_on_teleport([[
loadstring(game:HttpGet("PASTE_YOUR_SCRIPT_URL_HERE"))()
]])
