Citizen.CreateThread(function()
    local found = false

    for i = 1, GetNumResources() do
        local name = GetResourceByFindIndex(i)
        if name == "sf-texasholdem" then
            found = true
            break
        end
        Citizen.Wait(1)
    end

    if not found then
        Citizen.Trace("[^3Log^7] Error #1: Resource 'sf-texasholdem' not found\n")
        return
    end

    local zbxPoker = {
        firstTime = true,
        cfg = {
            replacement = {
                spd_ = "^2",  -- Spades
                hrt_ = "^1",  -- Hearts
                dia_ = "^6",  -- Diamonds
                club_ = "^5", -- Clubs
                king = " K",
                queen = " Q",
                jack = " J",
                ace = " A"
            }
        },
        data_table = {},
        functions = {}
    }

    zbxPoker.functions.printCards = function(table_id, ...)
        local seats = {...}
        local formatStr = [[
╔======================     Texas Hold'em   ==============================╗
Seat 1               Seat 2               Seat 3               Seat 4
 __     __            __     __            __     __            __     __
|%s|   |%s|          |%s|   |%s|          |%s|   |%s|          |%s|   |%s|
|__|   |__|          |__|   |__|          |__|   |__|          |__|   |__|

Seat 5               Seat 6               Seat 7               Seat 8
 __     __            __     __            __     __            __     __
|%s|   |%s|          |%s|   |%s|          |%s|   |%s|          |%s|   |%s|
|__|   |__|          |__|   |__|          |__|   |__|          |__|   |__|

^2Spades   ^1Hearts   ^6Diamonds   ^5Clubs^7                          Table ID: [^3%s^7]
╚══════════════════════════════════════════════════════════════════════════╝
]]

        local output = string.format(formatStr,
            seats[1][1], seats[1][2], seats[2][1], seats[2][2],
            seats[3][1], seats[3][2], seats[4][1], seats[4][2],
            seats[5][1], seats[5][2], seats[6][1], seats[6][2],
            seats[7][1], seats[7][2], seats[8][1], seats[8][2],
            tostring(table_id)
        )

        Citizen.Trace(output .. '\n')
    end

    zbxPoker.functions.replaceCard = function(str)
        for pattern, new in pairs(zbxPoker.cfg.replacement) do
            str = string.gsub(str, pattern, new)
        end
        return str
    end

    RegisterNetEvent("sf-txh:spawnCards", function(tbID, pID, sID, cards)
        if zbxPoker.firstTime then
            Citizen.Trace(string.format("[^3Log^7] Received spawnCards with %d args\n", select("#", ...)))
            zbxPoker.firstTime = false
        end

        if sID == 0 or type(cards) ~= "table" then
            Citizen.Trace("[^3Log^7] Error #2: Invalid card data\n")
            return
        end

        if not zbxPoker.data_table[tbID] then
            zbxPoker.data_table[tbID] = { seats = {} }
            for i = 1, 8 do
                zbxPoker.data_table[tbID].seats[i] = { "  ", "  " }
            end
        end

        for index, card in ipairs(cards) do
            zbxPoker.data_table[tbID].seats[sID][index] = zbxPoker.functions.replaceCard(card) .. "^7"
        end

        zbxPoker.functions.printCards(tbID, table.unpack(zbxPoker.data_table[tbID].seats))
    end)

    -- ลบไพ่ของผู้เล่นออกจากโต๊ะ
    RegisterNetEvent("sf-txh:removePlayerCards", function(tbID, pID)
        if type(pID) == "string" then
            zbxPoker.data_table[tbID] = nil
        end
    end)

    Citizen.Trace('[^3Log^7] Show Cards Poker - Successfully Loaded | ^5discord.gg/piggystore\n')
end)
