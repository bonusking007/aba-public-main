repeat task.wait(0.1) until game:IsLoaded()

-- ===== CONFIG =====
getgenv().main = true
getgenv().alt  = false

getgenv().MainAccounts = {"Onett10i83", "DITR08394", "GodJirazo83641"}
getgenv().AltAccounts  = {"Yoko_7822", "IbukiGames8927", "keiichi3742", "coconaphoenix", "ErikoShadow2413", "Shingo_3587", "Gardirime87348", "Laisbeppu11284", "Musatvizzi3621", "abafarmer96877567", "abafarmer912747567", "RicefarmerGrand1893", "grandfarmer357215", "Minesonos8632"}
-- ==================

setfpscap(30)
local Players = game:GetService("Players")
local VirtualInputManager = game:GetService("VirtualInputManager")
local VirtualUser = game:GetService("VirtualUser")
local HttpService = game:GetService("HttpService")
local UIS = game:GetService("UserInputService")
local LP = Players.LocalPlayer

local isRoundEnding = false

-- MAIN: ตรวจว่ามี alt ในเซิฟ ถ้าไม่มีให้ hop
task.spawn(function()
	if not getgenv().main then return end
	task.wait(5)
	local function checkAlts()
		local found = false
		for _, alt in ipairs(getgenv().AltAccounts) do
			if Players:FindFirstChild(alt) then found = true break end
		end
		if not found then
			-- ตรวจซ้ำ 3 ครั้งก่อน hop
			local confirm = 0
			for _ = 1, 3 do
				task.wait(5)
				local recheck = false
				for _, alt in ipairs(getgenv().AltAccounts) do
					if Players:FindFirstChild(alt) then recheck = true break end
				end
				if recheck then
					warn("[WWHub] Alt found on recheck — staying")
					return
				end
				confirm += 1
			end
			warn("[WWHub] Alt not found after recheck — hopping...")
			pcall(function() game:GetService("TeleportService"):TeleportToRandomPlace(game.PlaceId) end)
		else
			warn("[WWHub] Alt found — staying")
		end
	end
	checkAlts()
	while true do
		task.wait(60 * 30)
		if not getgenv().main then break end
		checkAlts()
	end
end)

-- ALT: ตรวจว่ามี main ในเซิฟ ถ้าไม่เจอใน 10 นาทีให้ hop
task.spawn(function()
	if not getgenv().alt then return end
	task.wait(5)
	local function checkMain()
		for _, name in ipairs(getgenv().MainAccounts) do
			if Players:FindFirstChild(name) then return true end
		end
		return false
	end
	local found = false
	for _ = 1, 60 do
		if checkMain() then found = true break end
		task.wait(10)
	end
	if not found then
		warn("[WWHub] Main not found after 10min — hopping...")
		pcall(function() game:GetService("TeleportService"):TeleportToRandomPlace(game.PlaceId) end)
		return
	end
	while true do
		task.wait(10)
		if not getgenv().alt then break end
		if not checkMain() then
			warn("[WWHub] Main left server — waiting 10min before hop...")
			local came_back = false
			for _ = 1, 60 do
				task.wait(10)
				if checkMain() then came_back = true break end
			end
			if not came_back then
				warn("[WWHub] Main still gone — hopping...")
				pcall(function() game:GetService("TeleportService"):TeleportToRandomPlace(game.PlaceId) end)
				break
			else
				warn("[WWHub] Main came back — staying")
			end
		end
	end
end)

local altCFrame        = CFrame.new(20000, 2000, 20000)
local pauseCFrame      = CFrame.new(156, 1, -43)
local pauseCFrameAlt   = CFrame.new(36, 5, -13)
local baseName         = "WWHub_BasePlate"

local tpDistanceLimit      = 18
local safeZoneLimit        = 8
local tpWatchdogDelay      = 0.35

local m1Hit = CFrame.new(
	97.64178466796875, 497.5, -602.8313598632812,
	0.9989567399024963, 0.006808227859437466, -0.045158419758081436,
	4.656613428188905e-10, 0.9888255000114441, 0.14907847344875336,
	0.04566875472664833, -0.14892295002937317, 0.9877936840057373
)

local loopAlt          = false
local loopMain         = false
local jugResetting     = false
local starting         = false
local selectingTeam    = false
local pointsCapped     = false
local roundPaused      = false
local roundPauseReason = nil
local roundResetting   = false
local handledChar      = nil
local pressedKChar     = nil
local timerTpDone      = false
local gui              = nil

local pointCapLimit = 100000

local _request
if not pcall(function()
	_request = request or http_request or http.request
end) then
	_request = function() end
end

local WebhookURL = "https://discord.com/api/webhooks/1453628734090514533/ddACObJX5Iuv966TcspBAEmkd5Er2ZfiVCMdoHzyONWLJ1CoqlDaAn3vg9D1GiZkvPoR"

local cachedAvatarUrl = ""
task.spawn(function()
	pcall(function()
		local apiUrl = string.format(
			"https://thumbnails.roblox.com/v1/users/avatar-headshot?userIds=%d&size=420x420&format=Png&isCircular=false",
			LP.UserId
		)
		local resp = HttpService:JSONDecode(game:HttpGet(apiUrl))
		if resp and resp.data and resp.data[1] and resp.data[1].imageUrl then
			cachedAvatarUrl = resp.data[1].imageUrl
		else
			cachedAvatarUrl = string.format(
				"https://www.roblox.com/headshot-thumbnail/image?userId=%d&width=420&height=420&format=png",
				LP.UserId
			)
		end
	end)
end)

local function sendWebhook(label)
	task.spawn(function()
		pcall(function()
			local moneyText, lvlText, pts, altCount = "N/A", "N/A", "N/A", 0
			pcall(function()
				moneyText = tostring(LP:WaitForChild("ReplicatedStats"):WaitForChild("Gold").Value)
				local hud = LP.PlayerGui:FindFirstChild("HUD")
				if hud then
					local lvlObj = hud:FindFirstChild("RightBotCorner")
						and hud.RightBotCorner:FindFirstChild("Line2")
						and hud.RightBotCorner.Line2:FindFirstChild("Lvl")
					if lvlObj then lvlText = lvlObj.Text end
				end
				pts = tostring(LP.leaderstats.Points.Value)
			end)
			for _, alt in ipairs(getgenv().AltAccounts) do
				if Players:FindFirstChild(alt) then altCount += 1 end
			end
			local altNames = {}
			for _, alt in ipairs(getgenv().AltAccounts) do
				if Players:FindFirstChild(alt) then
					table.insert(altNames, alt)
				end
			end
			local altList = #altNames > 0 and table.concat(altNames, "\n") or "None"
			_request({
				Url    = WebhookURL,
				Method = "POST",
				Headers = { ["Content-Type"] = "application/json" },
				Body = HttpService:JSONEncode({
					username   = "WW Hub",
					avatar_url = cachedAvatarUrl,
					embeds = {{
						title  = "⚡ WWHub — " .. label,
						color  = 0x46C864,
						thumbnail = { url = cachedAvatarUrl },
						fields = {
							{ name = "👤 Player", value = LP.DisplayName .. " (@" .. LP.Name .. ")", inline = false },
							{ name = "💰 Money",  value = moneyText, inline = true },
							{ name = "⭐ Level",  value = lvlText,   inline = true },
							{ name = "🎯 Points", value = pts,       inline = true },
							{ name = "👥 Alts", value = altCount .. " / " .. #getgenv().AltAccounts, inline = true },
							{ name = "📋 Alt Names", value = altList, inline = false },
						},
						footer = { text = "WWHub • " .. os.date("%d/%m/%Y %H:%M:%S") },
						timestamp = os.date("!%Y-%m-%dT%H:%M:%SZ")
					}}
				})
			})
		end)
	end)
end

-- ===== Helpers =====
local function getChar() return LP.Character end
local function getHRP()
	local c = getChar()
	return c and c:FindFirstChild("HumanoidRootPart")
end
local function getInput()
	local bp = LP:FindFirstChild("Backpack")
	local c  = getChar()
	return (bp and bp:FindFirstChild("Input"))
		or (c  and  c:FindFirstChild("Input"))
		or LP:FindFirstChild("Input")
end
local function fireInput(action, data)
	local inp = getInput()
	if not inp then return end
	pcall(function()
		if data then inp:FireServer(action, data)
		else inp:FireServer(action) end
	end)
end

local function tpToUntilArrived(cf, limit, maxTries)
	limit    = limit    or safeZoneLimit
	maxTries = maxTries or 20
	task.spawn(function()
		for _ = 1, maxTries do
			if not gui or not gui.Parent then break end
			local hrp = getHRP()
			if hrp then
				pcall(function()
					hrp.AssemblyLinearVelocity  = Vector3.new(0,0,0)
					hrp.AssemblyAngularVelocity = Vector3.new(0,0,0)
					hrp.CFrame = cf
				end)
				task.wait(0.25)
				hrp = getHRP()
				if hrp and (hrp.Position - cf.Position).Magnitude <= limit then
					break
				end
			end
			task.wait(0.25)
		end
	end)
end

local function tpToSafeZone()
	local cf = loopAlt and pauseCFrameAlt or pauseCFrame
	tpToUntilArrived(cf, safeZoneLimit, 20)
end

local function isNearCFrame(cf, limit)
	local hrp = getHRP()
	if not hrp or not cf then return false end
	return (hrp.Position - cf.Position).Magnitude <= (limit or tpDistanceLimit)
end
local function isNearFarm() return isNearCFrame(altCFrame, tpDistanceLimit) end

local function makeBase()
	local base = workspace:FindFirstChild(baseName)
	if not base then
		base          = Instance.new("Part")
		base.Name     = baseName
		base.Anchored = true
		base.Color    = Color3.fromRGB(35,35,35)
		base.Material = Enum.Material.SmoothPlastic
		base.Parent   = workspace
	end
	base.Size   = Vector3.new(500, 0.4, 500)
	base.CFrame = CFrame.new(altCFrame.Position - Vector3.new(0, 3, 0))
	base.Anchored = true
end

local function tpToCFrame(cf)
	if not cf then return false end
	makeBase()
	local hrp = getHRP()
	if not hrp then return false end
	pcall(function()
		hrp.AssemblyLinearVelocity  = Vector3.new(0,0,0)
		hrp.AssemblyAngularVelocity = Vector3.new(0,0,0)
		hrp.CFrame = cf
	end)
	return true
end
local function tpAndVerify(cf)
	if not tpToCFrame(cf) then return false end
	task.delay(tpWatchdogDelay, function()
		if gui and gui.Parent and not selectingTeam and (loopAlt or loopMain)
			and not isNearCFrame(cf, tpDistanceLimit)
			and not timerTpDone and not roundPaused then
			tpToCFrame(cf)
		end
	end)
	return true
end

local function getMainCFrame()
	local p = altCFrame.Position
	return CFrame.lookAt(p + Vector3.new(0,0,-1), p)
end
local function getFarmTargetCFrame()
	if loopAlt  then return altCFrame end
	if loopMain then return getMainCFrame() end
	return nil
end
local function tpAlt()  tpAndVerify(altCFrame) end
local function tpMain() tpAndVerify(getMainCFrame()) end

-- ===== Input / Skills =====
local function fireM1()
	fireInput("M1", {
		air=false, skeyreal=false, skeydown=true,
		mousehit=m1Hit, md=Vector3.new(0,0,0)
	})
end
local function pressKey(key)
	pcall(function()
		VirtualInputManager:SendKeyEvent(true,  key, false, game)
		task.wait(0.05)
		VirtualInputManager:SendKeyEvent(false, key, false, game)
	end)
end
local function fireSkills()
	local inp = getInput()
	if not inp then return end
	pcall(function() inp:FireServer("UseMove",{air=false,running=false,neutral=true,range="1",ToolName="Getsuga Tensho",mousehit=m1Hit,camdir=vector.create(-0.83,-0.065,-0.55),campos=vector.create(3083.8,579.3,473.5)}) end)
	pressKey(Enum.KeyCode.One)   task.wait(0.15)
	pcall(function() inp:FireServer("UseMove",{air=false,running=false,neutral=true,range="2",ToolName="Getsuga Slash",mousehit=m1Hit,camdir=vector.create(-0.82,-0.033,-0.57),campos=vector.create(3083.6,578.9,473.7)}) end)
	pressKey(Enum.KeyCode.Two)   task.wait(0.15)
	pcall(function() inp:FireServer("UseMove",{air=false,running=false,neutral=true,range="3",ToolName="Multi-Cut",mousehit=m1Hit,camdir=vector.create(-0.91,-0.095,-0.39),campos=vector.create(3047.1,579.7,438.5)}) end)
	pressKey(Enum.KeyCode.Three) task.wait(0.15)
	pcall(function() inp:FireServer("UseMove",{air=false,running=false,neutral=true,range="4",ToolName="Lunge",mousehit=m1Hit,camdir=vector.create(-0.75,-0.018,-0.65),campos=vector.create(3047.1,578.7,444.8)}) end)
	pressKey(Enum.KeyCode.Four)  task.wait(0.15)
	pcall(function() inp:FireServer("UseMode") end)
end
local function forceFieldOff() fireInput("ForceFieldOff") end
local function pressK()
	pcall(function() VirtualInputManager:SendKeyEvent(true, Enum.KeyCode.K, false, game) end)
end

task.spawn(function()
	while gui and gui.Parent do
		if loopMain and not roundPaused and not timerTpDone and not starting then
			pcall(function()
				VirtualInputManager:SendKeyEvent(true,  Enum.KeyCode.G, false, game)
				task.wait(0.03)
				VirtualInputManager:SendKeyEvent(false, Enum.KeyCode.G, false, game)
			end)
		end
		task.wait(0.05)
	end
end)

pcall(function()
	LP.Idled:Connect(function()
		pcall(function()
			VirtualUser:CaptureController()
			VirtualUser:Button2Down(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
			task.wait(0.1)
			VirtualUser:Button2Up(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
		end)
	end)
end)

-- ===== Character =====
local function fireRespawnDone()
	local pg = LP:WaitForChild("PlayerGui")
	local r  = pg:FindFirstChild("Respawning")
	if r and r:FindFirstChild("Done") then
		pcall(function() r.Done:FireServer() end)
	end
end
local function afterCharacterLoaded(char)
	if not char or handledChar == char then return end
	handledChar  = char
	pressedKChar = nil
	char:WaitForChild("HumanoidRootPart", 10)
	char:WaitForChild("Humanoid", 10)
	task.wait(1)
	fireRespawnDone()
	task.wait(0.2)
	forceFieldOff()
end
local function resetCharacter()
	local char = getChar()
	if char then
		local hum = char:FindFirstChildOfClass("Humanoid")
		if hum then hum.Health = 0 end
	end
	pcall(function() game:GetService("ReplicatedStorage"):WaitForChild("Loaded"):FireServer() end)
	task.wait(1)
	local nc = getChar()
	if nc then afterCharacterLoaded(nc) end
end
local function pressKAfterTP()
	local char = LP.Character
	if not char or pressedKChar == char or not isNearFarm() then return end
	pressedKChar = char
	task.wait(0.35)
	if isNearFarm() then pressK() end
end

-- ===== Team =====
local function getTeamPad()
	if loopAlt  then return workspace:FindFirstChild("Blue Team") end
	if loopMain then return workspace:FindFirstChild("Red Team")  end
	return nil
end
local function selectTeam()
	if selectingTeam then return end
	selectingTeam = true
	local pad = getTeamPad()
	if not pad then selectingTeam = false return end
	while workspace:FindFirstChild(pad.Name) and not roundPaused do
		local hrp = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
		if hrp then
			local pp = pad:IsA("BasePart") and pad or pad:FindFirstChildWhichIsA("BasePart")
			if pp then hrp.CFrame = pp.CFrame + Vector3.new(0,3,0) end
		end
		task.wait(0.1)
	end
	task.wait(0.5)
	selectingTeam = false
end

-- ===== Stats =====
local function getPoints()
	local ok, v = pcall(function() return LP.leaderstats.Points.Value end)
	return ok and v or 0
end
local function getTimerValue()
	local ok, v = pcall(function()
		local hud = LP.PlayerGui:FindFirstChild("HUD")
		if not hud then return 0 end
		local t = hud:FindFirstChild("Timer")
		if not t then return 0 end
		if t:IsA("TextLabel") or t:IsA("TextBox") then
			return tonumber(t.Text) or 0
		else
			return tonumber(t.Value) or 0
		end
	end)
	return ok and v or 0
end
local function getLives()
	local ok, v = pcall(function()
		local hud = LP.PlayerGui:FindFirstChild("HUD")
		if not hud then return 0 end
		local sc = hud:FindFirstChild("StockCount")
		if not sc then return 0 end
		return tonumber(sc.Text:match("%d+")) or 0
	end)
	return ok and v or 0
end

-- ===== Blocked Mode =====
local function getBlockedFarmMode()
	local pg = LP:FindFirstChild("PlayerGui")
	if not pg then return nil end
	local lb = pg:FindFirstChild("CustomLeaderboard")
	if not lb then return nil end
	local m = lb:FindFirstChild("Main") or lb
	if m:FindFirstChild("Juggernaut", true) then return "Juggernaut" end
	if m:FindFirstChild("Lives",      true) then return "Lives"      end
	for _, obj in ipairs(m:GetDescendants()) do
		local nm = string.lower(obj.Name or "")
		if nm:find("juggernaut") then return "Juggernaut" end
		if nm:find("lives")      then return "Lives"      end
		if obj:IsA("TextLabel") or obj:IsA("TextBox") or obj:IsA("TextButton") then
			local tx = string.lower(obj.Text or "")
			if tx:find("juggernaut") then return "Juggernaut" end
			if tx:find("lives")      then return "Lives"      end
		end
	end
	return nil
end

local function drainLives()
	print("[WWHub] Draining lives until round ends...")
	while gui and gui.Parent and (loopMain or loopAlt) do
		if not getBlockedFarmMode() then break end
		local char = getChar()
		if char then
			local hum = char:FindFirstChildOfClass("Humanoid")
			if hum and hum.Health > 0 then hum.Health = 0 end
		end
		task.wait(2)
		local nc = getChar()
		if nc then afterCharacterLoaded(nc) end
		task.wait(0.5)
		print("[WWHub] Lives left: " .. getLives())
	end
	print("[WWHub] Round ended — resuming farm")
end

local function pauseFarmForRound(reason)
	if roundPaused then return end
	roundPaused      = true
	roundPauseReason = reason or "Blocked Mode"
	pointsCapped     = false
	sendWebhook("Farm Paused — " .. roundPauseReason)
	if roundResetting then return end
	roundResetting = true
	task.spawn(function()
		drainLives()
		tpToSafeZone()
		local clear = 0
		while gui and gui.Parent and (loopMain or loopAlt) and roundPaused do
			local bm = getBlockedFarmMode()
			if bm then
				clear = 0
				tpToSafeZone()
				task.wait(1)
			else
				clear += 1
				if clear >= 3 then break end
				task.wait(1)
			end
		end
		local last = roundPauseReason or "Blocked Mode"
		roundPaused      = false
		roundPauseReason = nil
		roundResetting   = false
		if gui and gui.Parent and (loopMain or loopAlt) then
			sendWebhook("Farm Resumed — " .. last)
		end
	end)
end

-- ===== Timer watchdog =====
task.spawn(function()
	while true do
		task.wait(0.3)
		if loopMain or loopAlt then
			local pts   = getPoints()
			local timer = getTimerValue()

			if loopMain then
				if not pointsCapped and pts >= pointCapLimit then
					pointsCapped = true
				end
				if pointsCapped and timer > 30 and pts < pointCapLimit then
					pointsCapped = false
				end
			end

			-- timer ≤ 2 → หยุดฟาร์ม + TP safezone + flag round ending
			if timer > 0 and timer <= 2 and not timerTpDone and not roundPaused then
				timerTpDone    = true
				isRoundEnding  = true
				tpToSafeZone()
				-- reset flag หลัง 20 วิ (โหลดตาใหม่เสร็จแล้ว)
				task.delay(20, function()
					isRoundEnding = false
				end)
			end

			if timerTpDone then
				local padExists = workspace:FindFirstChild("Red Team")
					or workspace:FindFirstChild("Blue Team")
					or workspace:FindFirstChild("Green Team")
				if timer > 30 or padExists then
					timerTpDone = false
				end
			end
		else
			pointsCapped  = false
			timerTpDone   = false
			isRoundEnding = false
		end
	end
end)

-- ===== startFarm =====
local startFarm
startFarm = function(mode)
	if starting then return end
	starting = true
	loopAlt  = false
	loopMain = false
	makeBase()
	local function selectIchigoPlay()
		fireInput("CharacterButton", "Ichigo")
		task.wait(0.2)
		fireInput("ClickPlay")
	end
	selectIchigoPlay()
	task.wait(2.5)
	resetCharacter()
	task.wait(2.5)
	selectIchigoPlay()
	task.wait(2.5)
	roundPaused      = false
	roundPauseReason = nil
	roundResetting   = false
	timerTpDone      = false
	pointsCapped     = false
	isRoundEnding    = false
	if mode == "alt" then loopAlt  = true
	else                  loopMain = true end
	if mode == "main" then sendWebhook("Main Farm Started") end
	starting = false
end

-- ===== GUI =====
gui = Instance.new("ScreenGui")
gui.Name          = "WWHub_GUI_v4"
gui.ResetOnSpawn  = false
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.Parent        = game.CoreGui

local toggleBtn = Instance.new("TextButton")
toggleBtn.Size             = UDim2.new(0, 42, 0, 42)
toggleBtn.Position         = UDim2.new(0, 10, 0.5, -21)
toggleBtn.BackgroundColor3 = Color3.fromRGB(18, 18, 28)
toggleBtn.Text             = "⚡"
toggleBtn.TextSize         = 20
toggleBtn.TextColor3       = Color3.fromRGB(150, 70, 255)
toggleBtn.Font             = Enum.Font.GothamBold
toggleBtn.ZIndex           = 10
toggleBtn.Parent           = gui
Instance.new("UICorner", toggleBtn).CornerRadius = UDim.new(0, 10)
local ts = Instance.new("UIStroke", toggleBtn)
ts.Color = Color3.fromRGB(110, 40, 200); ts.Thickness = 2

local panel = Instance.new("Frame")
panel.Size             = UDim2.new(0, 280, 0, 320)
panel.Position         = UDim2.new(0.5, -140, 0.5, -160)
panel.BackgroundColor3 = Color3.fromRGB(14, 14, 22)
panel.BorderSizePixel  = 0
panel.Active           = true
panel.Parent           = gui
Instance.new("UICorner", panel).CornerRadius = UDim.new(0, 14)
local ps = Instance.new("UIStroke", panel)
ps.Color = Color3.fromRGB(100, 35, 190); ps.Thickness = 2

local dragging, dragStart, startPos
panel.InputBegan:Connect(function(inp)
	if inp.UserInputType == Enum.UserInputType.MouseButton1
	or inp.UserInputType == Enum.UserInputType.Touch then
		dragging = true; dragStart = inp.Position; startPos = panel.Position
	end
end)
panel.InputEnded:Connect(function(inp)
	if inp.UserInputType == Enum.UserInputType.MouseButton1
	or inp.UserInputType == Enum.UserInputType.Touch then
		dragging = false
	end
end)
if UIS then
	UIS.InputChanged:Connect(function(inp)
		if dragging and (inp.UserInputType == Enum.UserInputType.MouseMovement
		or inp.UserInputType == Enum.UserInputType.Touch) then
			local d = inp.Position - dragStart
			panel.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + d.X,
			                           startPos.Y.Scale, startPos.Y.Offset + d.Y)
		end
	end)
end

local header = Instance.new("Frame")
header.Size             = UDim2.new(1, 0, 0, 48)
header.BackgroundColor3 = Color3.fromRGB(20, 20, 32)
header.BorderSizePixel  = 0
header.Parent           = panel
Instance.new("UICorner", header).CornerRadius = UDim.new(0, 14)

local titleLbl = Instance.new("TextLabel")
titleLbl.Size               = UDim2.new(1, -50, 1, 0)
titleLbl.Position           = UDim2.new(0, 12, 0, 0)
titleLbl.BackgroundTransparency = 1
titleLbl.Text               = "⚡ WW Hub v4"
titleLbl.TextColor3         = Color3.fromRGB(155, 80, 255)
titleLbl.TextSize           = 19
titleLbl.Font               = Enum.Font.GothamBold
titleLbl.TextXAlignment     = Enum.TextXAlignment.Left
titleLbl.Parent             = header

local closeBtn = Instance.new("TextButton")
closeBtn.Size             = UDim2.new(0, 32, 0, 32)
closeBtn.Position         = UDim2.new(1, -40, 0, 8)
closeBtn.BackgroundColor3 = Color3.fromRGB(190, 35, 55)
closeBtn.Text             = "✕"
closeBtn.TextColor3       = Color3.fromRGB(255,255,255)
closeBtn.TextSize         = 15
closeBtn.Font             = Enum.Font.GothamBold
closeBtn.Parent           = header
Instance.new("UICorner", closeBtn).CornerRadius = UDim.new(0, 8)

local statusLbl = Instance.new("TextLabel")
statusLbl.Size               = UDim2.new(1, -20, 0, 20)
statusLbl.Position           = UDim2.new(0, 10, 0, 54)
statusLbl.BackgroundTransparency = 1
statusLbl.Text               = "📊 Idle"
statusLbl.TextColor3         = Color3.fromRGB(140,140,165)
statusLbl.TextSize           = 13
statusLbl.Font               = Enum.Font.Gotham
statusLbl.TextXAlignment     = Enum.TextXAlignment.Left
statusLbl.Parent             = panel

local div = Instance.new("Frame")
div.Size             = UDim2.new(1, -20, 0, 1)
div.Position         = UDim2.new(0, 10, 0, 78)
div.BackgroundColor3 = Color3.fromRGB(60, 35, 110)
div.BorderSizePixel  = 0
div.Parent           = panel

local function mkBtn(txt, col, yPos, w, xOff)
	local b = Instance.new("TextButton")
	b.Size             = UDim2.new(w or 1, -20, 0, 52)
	b.Position         = UDim2.new(0, xOff or 10, 0, yPos)
	b.BackgroundColor3 = col
	b.Text             = txt
	b.TextColor3       = Color3.fromRGB(255,255,255)
	b.TextSize         = 20
	b.Font             = Enum.Font.GothamBold
	b.Parent           = panel
	Instance.new("UICorner", b).CornerRadius = UDim.new(0, 10)
	return b
end

local altBtn  = mkBtn("💀 ALT",  Color3.fromRGB(50,110,240), 88, 0.47, 10)
local mainBtn = mkBtn("🎮 MAIN", Color3.fromRGB(50,185,90),  88, 0.47, nil)
mainBtn.Position = UDim2.new(0.53, -8, 0, 88)
local stopBtn = mkBtn("⏹ STOP", Color3.fromRGB(190,50,50), 150, 1, 10)

local capLbl = Instance.new("TextLabel")
capLbl.Size               = UDim2.new(1,-20,0,20)
capLbl.Position           = UDim2.new(0,10,0,212)
capLbl.BackgroundTransparency = 1
capLbl.Text               = "🎯 Point Cap: " .. pointCapLimit
capLbl.TextColor3         = Color3.fromRGB(255,200,60)
capLbl.TextSize           = 13
capLbl.Font               = Enum.Font.GothamBold
capLbl.TextXAlignment     = Enum.TextXAlignment.Left
capLbl.Parent             = panel

local function mkSmallBtn(txt, xOff)
	local b = Instance.new("TextButton")
	b.Size             = UDim2.new(0,36,0,28)
	b.Position         = UDim2.new(1, xOff, 0, 208)
	b.BackgroundColor3 = Color3.fromRGB(40,40,55)
	b.Text             = txt
	b.TextColor3       = Color3.fromRGB(255,255,255)
	b.TextSize         = 18
	b.Font             = Enum.Font.GothamBold
	b.Parent           = panel
	Instance.new("UICorner", b).CornerRadius = UDim.new(0,7)
	return b
end
local capMinus = mkSmallBtn("−", -90)
local capPlus  = mkSmallBtn("+", -50)

local renderEnabled = true
local renderBtn = mkBtn("👁 Render: ON", Color3.fromRGB(0,120,210), 248, 1, 10)
renderBtn.TextSize = 15

local destroyBtn = Instance.new("TextButton")
destroyBtn.Size             = UDim2.new(1,-20,0,30)
destroyBtn.Position         = UDim2.new(0,10,1,-38)
destroyBtn.BackgroundColor3 = Color3.fromRGB(35,35,50)
destroyBtn.Text             = "❌ Destroy GUI"
destroyBtn.TextColor3       = Color3.fromRGB(200,200,220)
destroyBtn.TextSize         = 13
destroyBtn.Font             = Enum.Font.GothamBold
destroyBtn.Parent           = panel
Instance.new("UICorner", destroyBtn).CornerRadius = UDim.new(0,8)

local function setStatus(txt) statusLbl.Text = "📊 " .. txt end

toggleBtn.MouseButton1Click:Connect(function()
	panel.Visible = not panel.Visible
end)
altBtn.MouseButton1Click:Connect(function()
	setStatus("Starting ALT...")
	task.spawn(function() startFarm("alt") end)
	altBtn.BackgroundColor3  = Color3.fromRGB(40,190,100); altBtn.Text  = "⏸ ALT"
	mainBtn.BackgroundColor3 = Color3.fromRGB(50,185,90);  mainBtn.Text = "🎮 MAIN"
end)
mainBtn.MouseButton1Click:Connect(function()
	setStatus("Starting MAIN...")
	task.spawn(function() startFarm("main") end)
	mainBtn.BackgroundColor3 = Color3.fromRGB(40,190,100); mainBtn.Text = "⏸ MAIN"
	altBtn.BackgroundColor3  = Color3.fromRGB(50,110,240); altBtn.Text  = "💀 ALT"
end)
stopBtn.MouseButton1Click:Connect(function()
	loopAlt = false; loopMain = false
	altBtn.BackgroundColor3  = Color3.fromRGB(50,110,240); altBtn.Text  = "💀 ALT"
	mainBtn.BackgroundColor3 = Color3.fromRGB(50,185,90);  mainBtn.Text = "🎮 MAIN"
	setStatus("Idle")
end)
capMinus.MouseButton1Click:Connect(function()
	pointCapLimit = math.max(10000, pointCapLimit - 10000)
	capLbl.Text = "🎯 Point Cap: " .. pointCapLimit
end)
capPlus.MouseButton1Click:Connect(function()
	pointCapLimit = pointCapLimit + 10000
	capLbl.Text = "🎯 Point Cap: " .. pointCapLimit
end)
renderBtn.MouseButton1Click:Connect(function()
	renderEnabled = not renderEnabled
	game:GetService("RunService"):Set3dRenderingEnabled(renderEnabled)
	renderBtn.Text             = renderEnabled and "👁 Render: ON" or "👁 Render: OFF"
	renderBtn.BackgroundColor3 = renderEnabled and Color3.fromRGB(0,120,210) or Color3.fromRGB(170,35,35)
end)
closeBtn.MouseButton1Click:Connect(function()
	panel.Visible = false
end)
destroyBtn.MouseButton1Click:Connect(function()
	loopAlt = false; loopMain = false
	gui:Destroy()
end)

task.spawn(function()
	task.wait(0.5)
	if getgenv().main then
		task.spawn(function() startFarm("main") end)
		mainBtn.BackgroundColor3 = Color3.fromRGB(40,190,100); mainBtn.Text = "⏸ MAIN"
	elseif getgenv().alt then
		task.spawn(function() startFarm("alt") end)
		altBtn.BackgroundColor3 = Color3.fromRGB(40,190,100); altBtn.Text = "⏸ ALT"
	end
end)

local function getLevel()
	local ok, v = pcall(function()
		local hud = LP.PlayerGui:FindFirstChild("HUD")
		if not hud then return "?" end
		local lvlObj = hud:FindFirstChild("RightBotCorner")
			and hud.RightBotCorner:FindFirstChild("Line2")
			and hud.RightBotCorner.Line2:FindFirstChild("Lvl")
		if lvlObj then
			return tostring(tonumber(lvlObj.Text:match("%d+")) or "?")
		end
		return "?"
	end)
	return ok and v or "?"
end

local function countAltsInServer()
	local count = 0
	for _, alt in ipairs(getgenv().AltAccounts) do
		if Players:FindFirstChild(alt) then count += 1 end
	end
	return count
end

local function isAltOnWrongTeam()
	if not loopAlt then return false end
	local char = getChar()
	if not char then return false end
	local ok, wrong = pcall(function()
		local team = LP.Team
		if team then
			local name = team.Name:lower()
			return name:find("red") ~= nil
		end
		return false
	end)
	return ok and wrong or false
end

local noClipEnabled = false
task.spawn(function()
	while gui.Parent do
		if loopMain and not roundPaused and not timerTpDone then
			if not noClipEnabled then noClipEnabled = true end
			local char = getChar()
			if char then
				for _, part in ipairs(char:GetDescendants()) do
					if part:IsA("BasePart") then part.CanCollide = false end
				end
				local hrp = getHRP()
				if hrp and hrp.Position.Y < -10 then
					pcall(function()
						hrp.CFrame = CFrame.new(hrp.Position.X, 5, hrp.Position.Z)
						hrp.AssemblyLinearVelocity = Vector3.new(0,0,0)
					end)
				end
			end
		else
			if noClipEnabled then
				noClipEnabled = false
				local char = getChar()
				if char then
					for _, part in ipairs(char:GetDescendants()) do
						if part:IsA("BasePart") then part.CanCollide = true end
					end
				end
			end
		end
		task.wait(0.1)
	end
end)

task.spawn(function()
	while gui.Parent do
		if loopAlt and not roundPaused and not timerTpDone then
			if isAltOnWrongTeam() then
				warn("[WWHub] Alt on wrong team (Red) — going to safezone")
				roundPaused      = true
				roundPauseReason = "Wrong Team"
				tpToSafeZone()
				repeat task.wait(1) until
					workspace:FindFirstChild("Blue Team") or
					workspace:FindFirstChild("Red Team") or
					not gui.Parent
				task.wait(2)
				roundPaused      = false
				roundPauseReason = nil
			end
		end
		task.wait(1)
	end
end)

task.spawn(function()
	while gui and gui.Parent do
		local altCount = countAltsInServer()
		local altStr = " | alt=" .. altCount .. "/" .. #getgenv().AltAccounts .. " | Lvl:" .. getLevel()
		if starting then
			statusLbl.TextColor3 = Color3.fromRGB(255, 200, 50)
			statusLbl.Font       = Enum.Font.GothamBold
			statusLbl.TextSize   = 13
			setStatus("Starting...")
		elseif roundPaused then
			statusLbl.TextColor3 = Color3.fromRGB(255, 80, 80)
			statusLbl.Font       = Enum.Font.GothamBold
			statusLbl.TextSize   = 13
			setStatus("⏸ " .. (roundPauseReason or "Paused") .. altStr)
		elseif timerTpDone then
			statusLbl.TextColor3 = Color3.fromRGB(255, 165, 0)
			statusLbl.Font       = Enum.Font.GothamBold
			statusLbl.TextSize   = 13
			setStatus("⏱ Safe zone" .. altStr)
		elseif pointsCapped then
			statusLbl.TextColor3 = Color3.fromRGB(255, 215, 0)
			statusLbl.Font       = Enum.Font.GothamBold
			statusLbl.TextSize   = 13
			setStatus("🎯 Cap! pts=" .. getPoints() .. altStr)
		elseif loopAlt then
			statusLbl.TextColor3 = Color3.fromRGB(80, 255, 140)
			statusLbl.Font       = Enum.Font.GothamBold
			statusLbl.TextSize   = 14
			setStatus("💀 ALT | t=" .. getTimerValue() .. altStr)
		elseif loopMain then
			statusLbl.TextColor3 = Color3.fromRGB(100, 200, 255)
			statusLbl.Font       = Enum.Font.GothamBold
			statusLbl.TextSize   = 14
			setStatus("🎮 MAIN | t=" .. getTimerValue() .. altStr)
		else
			statusLbl.TextColor3 = Color3.fromRGB(140, 140, 165)
			statusLbl.Font       = Enum.Font.Gotham
			statusLbl.TextSize   = 13
			setStatus("Idle" .. altStr)
		end
		task.wait(0.5)
	end
end)

-- ===== Farm Loops =====
task.spawn(function()
	makeBase()
	while gui.Parent do
		local char = LP.Character
		if char and char.Parent and char ~= handledChar then
			afterCharacterLoaded(char)
		end
		if not roundPaused and not timerTpDone then pressKAfterTP() end
		task.wait(0.25)
	end
end)

task.spawn(function()
	while gui.Parent do
		local pad = getTeamPad()
		if pad and not selectingTeam and not roundPaused and not timerTpDone and (loopAlt or loopMain) then
			task.spawn(selectTeam)
		end
		if not selectingTeam and not jugResetting and not starting and not roundPaused and not timerTpDone then
			local target = getFarmTargetCFrame()
			if target and not isNearCFrame(target, tpDistanceLimit) then
				tpAndVerify(target)
			elseif loopAlt  then tpAlt()
			elseif loopMain then tpMain() end
		end
		task.wait(0.08)
	end
end)

task.spawn(function()
	while gui.Parent do
		if loopMain and not jugResetting and not starting
			and not selectingTeam and not roundPaused
			and not pointsCapped and not timerTpDone then
			fireM1()
		end
		task.wait(0.12)
	end
end)

task.spawn(function()
	while gui.Parent do
		if loopMain and not jugResetting and not starting
			and not selectingTeam and not roundPaused
			and not pointsCapped and not timerTpDone then
			fireSkills()
		end
		task.wait(0.8)
	end
end)

task.spawn(function()
	while gui.Parent do
		if (loopMain or loopAlt) and not starting and not roundPaused then
			local bm = getBlockedFarmMode()
			if bm then pauseFarmForRound(bm) end
		end
		task.wait(0.5)
	end
end)

task.spawn(function()
	local modes = {"Kills Team","Team Battle","3 Teams","Free For All","Kills FFA"}
	while gui.Parent do
		if (loopAlt or loopMain) and not roundPaused then
			for _, mode in ipairs(modes) do
				fireInput("mode", mode)
				task.wait(0.3)
			end
		end
		task.wait(1)
	end
end)

task.spawn(function()
	while gui.Parent do
		task.wait(30)
		if loopMain and not roundPaused then
			sendWebhook("Main Farm Report")
		end
	end
end)

-- ===== Rejoin (แก้แล้ว — ไม่ trigger ตอนจบตา) =====
local TeleportService = game:GetService("TeleportService")
local placeId = game.PlaceId

local function rejoin()
	if isRoundEnding then return end -- ป้องกัน rejoin ตอนจบตา
	pcall(function() sendWebhook("🔄 Rejoining — kicked/disconnected") end)
	task.wait(3)
	pcall(function() TeleportService:Teleport(placeId) end)
	task.wait(3)
	pcall(function() TeleportService:TeleportToRandomPlace(placeId) end)
end

LP.OnTeleport:Connect(function(state)
	if state == Enum.TeleportState.RequestedFromServer and not isRoundEnding then
		task.wait(3)
		rejoin()
	end
end)

-- AncestryChanged: เช็ค isRoundEnding ก่อน rejoin
LP.AncestryChanged:Connect(function(_, parent)
	if not parent and not isRoundEnding then
		rejoin()
	end
end)

pcall(function()
	LP.Kicked:Connect(function()
		if not isRoundEnding then rejoin() end
	end)
end)

-- Heartbeat watchdog: threshold 60 วิ + เช็ค isRoundEnding
task.spawn(function()
	local lastBeat = tick()
	game:GetService("RunService").Heartbeat:Connect(function()
		lastBeat = tick()
	end)
	while true do
		task.wait(15)
		if tick() - lastBeat > 60 and not isRoundEnding then
			if not Players:FindFirstChild(LP.Name) then
				rejoin()
				break
			else
				lastBeat = tick()
			end
		end
	end
end)
