--- V.6.3
repeat task.wait(0.1) until game:IsLoaded()
-- ===== CONFIG =====
_G.main  = {"zgvc14gryv82", "mggk74lotv86", "xmcojyxo4966"}
_G.alt   = {"zopwtlgh5396", "ukaxcnnc6062", "abfamxuz7891", "xjpf58gwwf37", "jzxbwdak3988", "rngb51qcbx72", "dxyzlqib1021", "dnzfqykq9893"}
_G.guard = {"vfzptjrf7587"}
-- ==================
setfpscap(25)
local Players    = game:GetService("Players")
local RunService = game:GetService("RunService")
local VIM        = game:GetService("VirtualInputManager")
local VUser      = game:GetService("VirtualUser")
local Http       = game:GetService("HttpService")
local UIS        = game:GetService("UserInputService")
local TS         = game:GetService("TeleportService")
local LP         = Players.LocalPlayer
local myName   = LP.Name
local IS_MAIN  = false
local IS_ALT   = false
local IS_GUARD = false
for _, n in ipairs(_G.main)  do if n == myName then IS_MAIN  = true end end
for _, n in ipairs(_G.alt)   do if n == myName then IS_ALT   = true end end
for _, n in ipairs(_G.guard) do if n == myName then IS_GUARD = true end end
warn("[WWHub] Role: " .. (IS_MAIN and "MAIN" or IS_ALT and "ALT" or IS_GUARD and "GUARD" or "UNKNOWN"))
-- ===== Server hop =====
task.spawn(function()
	if not IS_MAIN then return end
	task.wait(5)
	local function checkAlts()
		for _, n in ipairs(_G.alt) do if Players:FindFirstChild(n) then return true end end
		return false
	end
	local function hopIfNeeded()
		if checkAlts() then return end
		for _ = 1, 3 do task.wait(5) if checkAlts() then return end end
		pcall(function() TS:TeleportToRandomPlace(game.PlaceId) end)
	end
	hopIfNeeded()
	while true do task.wait(60*30) if IS_MAIN then hopIfNeeded() end end
end)
task.spawn(function()
	if not IS_ALT then return end
	task.wait(5)
	local function checkMain()
		for _, n in ipairs(_G.main) do if Players:FindFirstChild(n) then return true end end
		return false
	end
	local found = false
	for _ = 1, 60 do if checkMain() then found = true break end task.wait(10) end
	if not found then pcall(function() TS:TeleportToRandomPlace(game.PlaceId) end) return end
	while true do
		task.wait(10)
		if not IS_ALT then break end
		if not checkMain() then
			local back = false
			for _ = 1, 60 do task.wait(10) if checkMain() then back = true break end end
			if not back then pcall(function() TS:TeleportToRandomPlace(game.PlaceId) end) break end
		end
	end
end)
-- ===== Shared vars =====
local altCFrame      = CFrame.new(20000, 2000, 20000)
local pauseCFrame    = CFrame.new(156, 1, -43)
local pauseCFrameAlt = CFrame.new(36, 5, -13)
local baseName       = "WWHub_BasePlate"
local tpDistLimit    = 4
local safeLimit      = 8
local watchdogTime   = 0.35
local m1Hit = CFrame.new(
	97.64178466796875, 497.5, -602.8313598632812,
	0.9989567399024963, 0.006808227859437466, -0.045158419758081436,
	4.656613428188905e-10, 0.9888255000114441, 0.14907847344875336,
	0.04566875472664833, -0.14892295002937317, 0.9877936840057373
)
local loopAlt    = false
local loopMain   = false
local loopGuard  = false
local starting   = false
local selectingTeam = false
local pointsCapped  = false
local roundPaused   = false
local roundPauseReason = nil
local roundResetting   = false
local handledChar   = nil
local pressedKChar  = nil
local timerTpDone   = false
local gui           = nil
local pointCapLimit = 990000
local WebhookURL = "https://discord.com/api/webhooks/1453628734090514533/ddACObJX5Iuv966TcspBAEmkd5Er2ZfiVCMdoHzyONWLJ1CoqlDaAn3vg9D1GiZkvPoR"
