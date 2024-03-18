-- Crystal Conquest v.1
-- The core concept of the game revolves around players (or bots) collecting crystals
-- to gain power and eliminate each other. The game will progress towards a conclusion 
-- where the winner is determined by either who collects the most crystals within a set 
-- time frame or the last player standing.


-- Global Variables and Setup
GameMode = "Not-Started"
WaitTime = 120000 -- 2 minutes in milliseconds
GameTime = 600000 -- 10 minutes in milliseconds
Now = 0 -- This will be updated with the actual time

PaymentToken = "ADDR"
PaymentQty = 1000
BonusQty = 500
MinimumPlayers = 2

Players = {}
Waiting = {}
Crystals = {}
PlayerScores = {}

function initializeGame()
    GameMode = "Not-Started"
    Players = {}
    Waiting = {}
    Crystals = {}
    PlayerScores = {}
    distributeCrystals(10) -- Initial number of crystals
end

function distributeCrystals(numberOfCrystals)
    for i = 1, numberOfCrystals do
        local x = math.random(1, 50)
        local y = math.random(1, 50)
        Crystals[#Crystals + 1] = {x = x, y = y, collected = false}
    end
end

function startWaitingPeriod()
    GameMode = "Waiting"
    Now = os.time() * 1000 -- Update current time in milliseconds
    print("The game is about to begin! Please send your token to participate.")
end

function checkStartConditions()
    if #Waiting >= MinimumPlayers then
        startGame()
    else
        print("Not enough players registered! Waiting for more players...")
    end
end

function startGame()
    GameMode = "Playing"
    Now = os.time() * 1000 -- Update current time in milliseconds
    for playerId, _ in pairs(Waiting) do
        Players[playerId] = {x = math.random(1, 50), y = math.random(1, 50), score = 0}
        Waiting[playerId] = nil -- Remove player from waiting list
    end
    print("Game started. Good luck to all players!")
end

-- Initialization
initializeGame()
startWaitingPeriod()
function movePlayer(playerId, direction)
    local player = Players[playerId]
    if not player then
        print("Player not found.")
        return
    end

    -- Oyuncunun hareket yönünü belirle
    if direction == "up" then player.y = math.max(player.y - 1, 1)
    elseif direction == "down" then player.y = math.min(player.y + 1, 50)
    elseif direction == "left" then player.x = math.max(player.x - 1, 1)
    elseif direction == "right" then player.x = math.min(player.x + 1, 50)
    end

    -- Kristal toplama kontrolü
    for i, crystal in ipairs(Crystals) do
        if not crystal.collected and player.x == crystal.x and player.y == crystal.y then
            crystal.collected = true
            player.score = player.score + 1
            print("Player " .. playerId .. " collected a crystal at (" .. crystal.x .. ", " .. crystal.y .. "). Total score: " .. player.score)
            break
        end
    end
end

function attackPlayer(attackerId, defenderId)
    local attacker = Players[attackerId]
    local defender = Players[defenderId]

    if not attacker or not defender then
        print("Attack failed. One of the players does not exist.")
        return
    end

    -- Basit bir savaş mekanizması: Saldırganın skoru savunmacınınkinden fazlaysa, savunmacı elenir
    if attacker.score > defender.score then
        print("Player " .. defenderId .. " was eliminated by " .. attackerId)
        Players[defenderId] = nil -- Savunmacıyı oyuncu listesinden çıkar
    else
        print("Attack failed. Defender's score is equal to or higher than attacker's score.")
    end
end
function checkGameOver()
    if GameMode ~= "Playing" then return end

    local currentTime = os.time() * 1000
    if currentTime >= (Now + GameTime) then
        endGame()
    end
end

function endGame()
    print("Game Over. Calculating scores and determining the winner...")

    local highestScore = 0
    local winners = {}
    for playerId, player in pairs(Players) do
        if player.score > highestScore then
            highestScore = player.score
            winners = {playerId}
        elseif player.score == highestScore then
            table.insert(winners, playerId)
        end
    end

    if #winners == 1 then
        print("Winner: Player " .. winners[1] .. " with " .. highestScore .. " crystals.")
        sendReward(winners[1], BonusQty, "Winning the game")
    else
        print("It's a tie between players: " .. table.concat(winners, ", ") .. ". Each with " .. highestScore .. " crystals.")
        -- Tie-breaker logic could be implemented here
    end

    initializeGame() -- Oyunu yeniden başlat
end

-- Bu fonksiyonun düzenli olarak çağrılması, oyunun sonlanma koşullarını kontrol eder
-- checkGameOver() fonksiyonunun, oyun döngüsü içinde veya belirli aralıklarla çağrılması gerekir
