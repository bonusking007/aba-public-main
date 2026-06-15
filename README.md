--- V.6.2
repeat task.wait(0.1) until game:IsLoaded()

-- ===== CONFIG =====
_G.main  = {"igll89dwjm52", "ephe53qzzu56", "Shuhua_Ping"}
_G.alt   = {"ojexrppy9770", "rwfi55ngxj28", "vgakarhu6240", "ibdm14ljog99", "bnevporw3273", "mhqdcvee3722", "dagasvqp5610", "Laisbeppu11284", "Musatvizzi3621", "abafarmer96877567", "abafarmer912747567", "RicefarmerGrand1893", "grandfarmer357215", "Minesonos8632"}
_G.guard = {"Gaeul_4122", "qaaxvbyw5047"}
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
local tpDistLimit    = 18
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
local _request
if not pcall(function() _request = request or http_request or http.request end) then
	_request = function() end
end

local avatarUrl = ""
task.spawn(function()
	pcall(function()
		local url = string.format("https://thumbnails.roblox.com/v1/users/avatar-headshot?userIds=%d&size=420x420&format=Png&isCircular=false", LP.UserId)
		local r = Http:JSONDecode(game:HttpGet(url))
		if r and r.data and r.data[1] then avatarUrl = r.data[1].imageUrl or "" end
	end)
end)

local function sendWebhook(label)
	task.spawn(function()
		pcall(function()
			local money, lvl, pts, altCount = "N/A","N/A","N/A",0
			pcall(function()
				money = tostring(LP:WaitForChild("ReplicatedStats"):WaitForChild("Gold").Value)
				local hud = LP.PlayerGui:FindFirstChild("HUD")
				if hud then
					local lo = hud:FindFirstChild("RightBotCorner") and hud.RightBotCorner:FindFirstChild("Line2") and hud.RightBotCorner.Line2:FindFirstChild("Lvl")
					if lo then lvl = lo.Text end
				end
				pts = tostring(LP.leaderstats.Points.Value)
			end)
			local altNames = {}
			for _, n in ipairs(_G.alt) do if Players:FindFirstChild(n) then altCount+=1 table.insert(altNames,n) end end
			_request({
				Url=WebhookURL, Method="POST",
				Headers={["Content-Type"]="application/json"},
				Body=Http:JSONEncode({
					username="WW Hub", avatar_url=avatarUrl,
					embeds={{
						title="⚡ WWHub — "..label, color=0x46C864,
						thumbnail={url=avatarUrl},
						fields={
							{name="👤 Player",value=LP.DisplayName.." (@"..LP.Name..")",inline=false},
							{name="💰 Money",value=money,inline=true},
							{name="⭐ Level",value=lvl,inline=true},
							{name="🎯 Points",value=pts,inline=true},
							{name="👥 Alts",value=altCount.."/"..#_G.alt,inline=true},
							{name="🛡 Guards",value=countGuards().."/"..#_G.guard,inline=true},
							{name="📋 Alt List",value=#altNames>0 and table.concat(altNames,"\n") or "None",inline=false},
						},
						footer={text="WWHub • "..os.date("%d/%m/%Y %H:%M:%S")},
						timestamp=os.date("!%Y-%m-%dT%H:%M:%SZ")
					}}
				})
			})
		end)
	end)
end

-- ===== Helpers =====
local function getChar()  return LP.Character end
local function getHRP()   local c=getChar() return c and c:FindFirstChild("HumanoidRootPart") end
local function getHum()   local c=getChar() return c and c:FindFirstChildOfClass("Humanoid") end
local function getInput()
	local bp=LP:FindFirstChild("Backpack") local c=getChar()
	return (bp and bp:FindFirstChild("Input")) or (c and c:FindFirstChild("Input")) or LP:FindFirstChild("Input")
end
local function fireInput(action, data)
	local inp=getInput() if not inp then return end
	pcall(function() if data then inp:FireServer(action,data) else inp:FireServer(action) end end)
end
local function isCharReady()
	local c=getChar() if not c or not c.Parent then return false end
	local hrp=c:FindFirstChild("HumanoidRootPart")
	local hum=c:FindFirstChildOfClass("Humanoid")
	if not hrp or not hum then return false end
	if hum.Health<=0 then return false end
	if hrp.Position.Y<-100 then return false end
	return true
end
local function makeBase()
	local b=workspace:FindFirstChild(baseName)
	if not b then
		b=Instance.new("Part") b.Name=baseName b.Anchored=true
		b.Color=Color3.fromRGB(35,35,35) b.Material=Enum.Material.SmoothPlastic b.Parent=workspace
	end
	b.Size=Vector3.new(500,0.4,500)
	b.CFrame=CFrame.new(altCFrame.Position-Vector3.new(0,3,0))
	b.Anchored=true
end
local function tpToUntilArrived(cf,limit,tries)
	limit=limit or safeLimit tries=tries or 20
	task.spawn(function()
		for _=1,tries do
			if not gui or not gui.Parent then break end
			local hrp=getHRP()
			if hrp then
				pcall(function()
					hrp.AssemblyLinearVelocity=Vector3.new(0,0,0)
					hrp.AssemblyAngularVelocity=Vector3.new(0,0,0)
					hrp.CFrame=cf
				end)
				task.wait(0.25) hrp=getHRP()
				if hrp and (hrp.Position-cf.Position).Magnitude<=limit then break end
			end
			task.wait(0.25)
		end
	end)
end
local function tpToSafeZone()
	tpToUntilArrived(IS_ALT and pauseCFrameAlt or pauseCFrame, safeLimit, 20)
end
local function isNearCF(cf,limit)
	local hrp=getHRP() if not hrp or not cf then return false end
	return (hrp.Position-cf.Position).Magnitude<=(limit or tpDistLimit)
end
local function isNearFarm() return isNearCF(altCFrame,tpDistLimit) end
local function tpToCF(cf)
	if not cf then return false end makeBase()
	local hrp=getHRP() if not hrp then return false end
	pcall(function()
		hrp.AssemblyLinearVelocity=Vector3.new(0,0,0)
		hrp.AssemblyAngularVelocity=Vector3.new(0,0,0)
		hrp.CFrame=cf
	end)
	return true
end
local function tpAndVerify(cf)
	if not tpToCF(cf) then return false end
	task.delay(watchdogTime, function()
		if gui and gui.Parent and not selectingTeam and (loopAlt or loopMain)
			and not isNearCF(cf,tpDistLimit) and not timerTpDone and not roundPaused then
			tpToCF(cf)
		end
	end)
	return true
end
local function getMainCF()
	local p=altCFrame.Position return CFrame.lookAt(p+Vector3.new(0,0,-1),p)
end

-- ===== Skills =====
local function fireM1()
	fireInput("M1",{air=false,skeyreal=false,skeydown=true,mousehit=m1Hit,md=Vector3.new(0,0,0)})
end
local function pressKey(key)
	pcall(function() VIM:SendKeyEvent(true,key,false,game) task.wait(0.05) VIM:SendKeyEvent(false,key,false,game) end)
end
local function fireSkills()
	local inp=getInput() if not inp then return end
	pcall(function() inp:FireServer("UseMove",{air=false,running=false,neutral=true,range="1",ToolName="Getsuga Tensho",mousehit=m1Hit,camdir=vector.create(-0.83,-0.065,-0.55),campos=vector.create(3083.8,579.3,473.5)}) end)
	pressKey(Enum.KeyCode.One) task.wait(0.15)
	pcall(function() inp:FireServer("UseMove",{air=false,running=false,neutral=true,range="2",ToolName="Getsuga Slash",mousehit=m1Hit,camdir=vector.create(-0.82,-0.033,-0.57),campos=vector.create(3083.6,578.9,473.7)}) end)
	pressKey(Enum.KeyCode.Two) task.wait(0.15)
	pcall(function() inp:FireServer("UseMove",{air=false,running=false,neutral=true,range="3",ToolName="Multi-Cut",mousehit=m1Hit,camdir=vector.create(-0.91,-0.095,-0.39),campos=vector.create(3047.1,579.7,438.5)}) end)
	pressKey(Enum.KeyCode.Three) task.wait(0.15)
	pcall(function() inp:FireServer("UseMove",{air=false,running=false,neutral=true,range="4",ToolName="Lunge",mousehit=m1Hit,camdir=vector.create(-0.75,-0.018,-0.65),campos=vector.create(3047.1,578.7,444.8)}) end)
	pressKey(Enum.KeyCode.Four) task.wait(0.15)
	pcall(function() inp:FireServer("UseMode") end)
end
local function pressG()
	pcall(function() VIM:SendKeyEvent(true,Enum.KeyCode.G,false,game) task.wait(0.03) VIM:SendKeyEvent(false,Enum.KeyCode.G,false,game) end)
end
local function pressK()
	pcall(function() VIM:SendKeyEvent(true,Enum.KeyCode.K,false,game) end)
end
local function forceFieldOff() fireInput("ForceFieldOff") end

pcall(function()
	LP.Idled:Connect(function()
		pcall(function()
			VUser:CaptureController()
			VUser:Button2Down(Vector2.new(0,0),workspace.CurrentCamera.CFrame)
			task.wait(0.1)
			VUser:Button2Up(Vector2.new(0,0),workspace.CurrentCamera.CFrame)
		end)
	end)
end)

-- ===== Character =====
local function fireRespawnDone()
	local pg=LP:WaitForChild("PlayerGui")
	local r=pg:FindFirstChild("Respawning")
	if r and r:FindFirstChild("Done") then pcall(function() r.Done:FireServer() end) end
end
local function afterCharLoaded(char)
	if not char or handledChar==char then return end
	handledChar=char pressedKChar=nil
	char:WaitForChild("HumanoidRootPart",10)
	char:WaitForChild("Humanoid",10)
	task.wait(1) fireRespawnDone() task.wait(0.2) forceFieldOff()
end
local function resetChar()
	local c=getChar()
	if c then local h=c:FindFirstChildOfClass("Humanoid") if h then h.Health=0 end end
	pcall(function() game:GetService("ReplicatedStorage"):WaitForChild("Loaded"):FireServer() end)
	task.wait(1) local nc=getChar() if nc then afterCharLoaded(nc) end
end
local function pressKAfterTP()
	local c=LP.Character
	if not c or pressedKChar==c or not isNearFarm() then return end
	pressedKChar=c task.wait(0.35) if isNearFarm() then pressK() end
end

-- ===== Alt HP Reset =====
-- ใช้หลายวิธีในการฆ่า character เพราะ hum.Health=0 อาจถูก server block
local function killCharacter()
	local char = getChar()
	if not char then return end
	local hum = char:FindFirstChildOfClass("Humanoid")
	if not hum then return end
	-- วิธี 1: set Health ตรงๆ
	pcall(function() hum.Health = 0 end)
	task.wait(0.1)
	if hum.Health > 0 then
		-- วิธี 2: FireServer Loaded (แจ้ง respawn)
		pcall(function() game:GetService("ReplicatedStorage"):WaitForChild("Loaded"):FireServer() end)
	end
end

task.spawn(function()
	while gui and gui.Parent do
		if loopAlt and not roundPaused and not timerTpDone and not starting then
			local hum = getHum()
			local hrp = getHRP()
			if hum and hrp then
				local maxHp = hum.MaxHealth
				local curHp = hum.Health
				local ratio = maxHp > 0 and (curHp / maxHp) or 1
				local alive = curHp > 0 and hrp.Position.Y > -100

				if alive and ratio < 0.9 then
					warn("[WWHub] Alt HP " .. math.floor(ratio*100) .. "% — resetting")
					killCharacter()
					-- รอ respawn จริงๆ
					local w = 0
					repeat task.wait(0.3) w += 0.3 until isCharReady() or w > 15
					local nc = getChar()
					if nc then afterCharLoaded(nc) end
					task.wait(0.5)
					if loopAlt and not roundPaused then tpToCF(altCFrame) end

				elseif not alive and curHp <= 0 then
					-- ตายแล้วรอ respawn
					local w = 0
					repeat task.wait(0.3) w += 0.3 until isCharReady() or w > 15
					local nc = getChar()
					if nc then afterCharLoaded(nc) end
					task.wait(0.5)
					if loopAlt and not roundPaused then tpToCF(altCFrame) end
				end
			end
		end
		task.wait(0.2)
	end
end)

-- ===== Team =====
local function getTeamPad()
	if loopAlt  then return workspace:FindFirstChild("Blue Team") end
	if loopMain then return workspace:FindFirstChild("Red Team")  end
	return nil
end
local function hasTeamPads()
	return workspace:FindFirstChild("Blue Team")~=nil
		or workspace:FindFirstChild("Red Team")~=nil
		or workspace:FindFirstChild("Green Team")~=nil
end
local function selectTeam()
	if selectingTeam then return end selectingTeam=true
	local pad=getTeamPad() if not pad then selectingTeam=false return end
	while workspace:FindFirstChild(pad.Name) and not roundPaused do
		local hrp=LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
		if hrp then
			local pp=pad:IsA("BasePart") and pad or pad:FindFirstChildWhichIsA("BasePart")
			if pp then hrp.CFrame=pp.CFrame+Vector3.new(0,3,0) end
		end
		task.wait(0.1)
	end
	task.wait(0.5) selectingTeam=false
end

-- ===== Stats =====
local function getPoints() local ok,v=pcall(function() return LP.leaderstats.Points.Value end) return ok and v or 0 end
local function getTimerValue()
	local ok,v=pcall(function()
		local hud=LP.PlayerGui:FindFirstChild("HUD") if not hud then return 0 end
		local t=hud:FindFirstChild("Timer") if not t then return 0 end
		if t:IsA("TextLabel") or t:IsA("TextBox") then return tonumber(t.Text) or 0
		else return tonumber(t.Value) or 0 end
	end)
	return ok and v or 0
end
local function getLives()
	local ok,v=pcall(function()
		local hud=LP.PlayerGui:FindFirstChild("HUD") if not hud then return 0 end
		local sc=hud:FindFirstChild("StockCount") if not sc then return 0 end
		return tonumber(sc.Text:match("%d+")) or 0
	end)
	return ok and v or 0
end
local function getLevel()
	local ok,v=pcall(function()
		local hud=LP.PlayerGui:FindFirstChild("HUD") if not hud then return "?" end
		local lo=hud:FindFirstChild("RightBotCorner") and hud.RightBotCorner:FindFirstChild("Line2") and hud.RightBotCorner.Line2:FindFirstChild("Lvl")
		if lo then return tostring(tonumber(lo.Text:match("%d+")) or "?") end return "?"
	end)
	return ok and v or "?"
end
local function countAlts()
	local n=0 for _,name in ipairs(_G.alt) do if Players:FindFirstChild(name) then n+=1 end end return n
end
local function countGuards()
	local n=0 for _,name in ipairs(_G.guard) do if Players:FindFirstChild(name) then n+=1 end end return n
end

-- ===== Blocked Mode =====
local function getBlockedMode()
	local pg=LP:FindFirstChild("PlayerGui") if not pg then return nil end
	local lb=pg:FindFirstChild("CustomLeaderboard") if not lb then return nil end
	local m=lb:FindFirstChild("Main") or lb
	if m:FindFirstChild("Juggernaut",true) then return "Juggernaut" end
	if m:FindFirstChild("Lives",true) then return "Lives" end
	for _,obj in ipairs(m:GetDescendants()) do
		local nm=(obj.Name or ""):lower()
		if nm:find("juggernaut") then return "Juggernaut" end
		if nm:find("lives") then return "Lives" end
		if obj:IsA("TextLabel") or obj:IsA("TextBox") or obj:IsA("TextButton") then
			local tx=(obj.Text or ""):lower()
			if tx:find("juggernaut") then return "Juggernaut" end
			if tx:find("lives") then return "Lives" end
		end
	end
	return nil
end
local function drainLives()
	while gui and gui.Parent and (loopMain or loopAlt) do
		if not getBlockedMode() then break end
		local c=getChar() if c then local h=c:FindFirstChildOfClass("Humanoid") if h and h.Health>0 then h.Health=0 end end
		task.wait(2) local nc=getChar() if nc then afterCharLoaded(nc) end task.wait(0.5)
	end
end
local function pauseFarm(reason)
	if roundPaused then return end
	roundPaused=true roundPauseReason=reason or "Blocked" pointsCapped=false
	sendWebhook("Farm Paused — "..roundPauseReason)
	if roundResetting then return end roundResetting=true
	task.spawn(function()
		drainLives() tpToSafeZone()
		local clear=0
		while gui and gui.Parent and (loopMain or loopAlt) and roundPaused do
			local bm=getBlockedMode()
			if bm then clear=0 tpToSafeZone() task.wait(1)
			else clear+=1 if clear>=3 then break end task.wait(1) end
		end
		local last=roundPauseReason
		roundPaused=false roundPauseReason=nil roundResetting=false
		if gui and gui.Parent and (loopMain or loopAlt) then sendWebhook("Farm Resumed — "..last) end
	end)
end

-- Timer watchdog
task.spawn(function()
	while true do
		task.wait(0.3)
		if loopMain or loopAlt then
			local pts=getPoints() local timer=getTimerValue()
			if loopMain then
				if not pointsCapped and pts>=pointCapLimit then pointsCapped=true end
				if pointsCapped and timer>30 and pts<pointCapLimit then pointsCapped=false end
			end
			if timer>0 and timer<=2 and not timerTpDone and not roundPaused then
				timerTpDone=true tpToSafeZone()
			end
			if timerTpDone then
				local pad=workspace:FindFirstChild("Red Team") or workspace:FindFirstChild("Blue Team") or workspace:FindFirstChild("Green Team")
				if timer>30 or pad then timerTpDone=false end
			end
		else pointsCapped=false timerTpDone=false end
	end
end)

-- startFarm
local startFarm
startFarm=function(mode)
	if starting then return end
	starting=true loopAlt=false loopMain=false loopGuard=false makeBase()
	local function selectChar()
		if mode=="alt" then fireInput("CharacterButton","Random")
		else fireInput("CharacterButton","Ichigo") end
		task.wait(0.2) fireInput("ClickPlay")
	end
	selectChar() task.wait(2.5) resetChar() task.wait(2.5) selectChar() task.wait(2.5)
	roundPaused=false roundPauseReason=nil roundResetting=false timerTpDone=false pointsCapped=false
	if mode=="alt"   then loopAlt=true
	elseif mode=="main"  then loopMain=true
	elseif mode=="guard" then loopGuard=true end
	if mode=="main" then sendWebhook("Main Farm Started") end
	starting=false
end

-- ===== GUARD =====
local function getTargets()
	local skip={} skip[myName]=true
	for _,n in ipairs(_G.main)  do skip[n]=true end
	for _,n in ipairs(_G.alt)   do skip[n]=true end
	for _,n in ipairs(_G.guard) do skip[n]=true end
	local t={}
	for _,p in ipairs(Players:GetPlayers()) do
		if not skip[p.Name] and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
			table.insert(t,p)
		end
	end
	return t
end

-- G spam main
task.spawn(function()
	while gui and gui.Parent do
		if loopMain and not roundPaused and not timerTpDone and not starting then pressG() end
		task.wait(0.05)
	end
end)

-- ===== GUI =====
gui=Instance.new("ScreenGui")
gui.Name="WWHub_GUI_v62" gui.ResetOnSpawn=false
gui.ZIndexBehavior=Enum.ZIndexBehavior.Sibling gui.DisplayOrder=0 gui.Parent=game.CoreGui

local toggleBtn=Instance.new("TextButton")
toggleBtn.Size=UDim2.new(0,42,0,42) toggleBtn.Position=UDim2.new(0,10,0.5,-21)
toggleBtn.BackgroundColor3=Color3.fromRGB(18,18,28) toggleBtn.Text="⚡" toggleBtn.TextSize=20
toggleBtn.TextColor3=Color3.fromRGB(150,70,255) toggleBtn.Font=Enum.Font.GothamBold
toggleBtn.ZIndex=10 toggleBtn.Parent=gui
Instance.new("UICorner",toggleBtn).CornerRadius=UDim.new(0,10)
local ts=Instance.new("UIStroke",toggleBtn) ts.Color=Color3.fromRGB(110,40,200) ts.Thickness=2

local panel=Instance.new("Frame")
panel.Size=UDim2.new(0,280,0,340) panel.Position=UDim2.new(0.5,-140,0.5,-170)
panel.BackgroundColor3=Color3.fromRGB(14,14,22) panel.BorderSizePixel=0 panel.Active=true panel.Parent=gui
Instance.new("UICorner",panel).CornerRadius=UDim.new(0,14)
local ps=Instance.new("UIStroke",panel) ps.Color=Color3.fromRGB(100,35,190) ps.Thickness=2

local dragging,dragStart,dragPos
panel.InputBegan:Connect(function(inp)
	if inp.UserInputType==Enum.UserInputType.MouseButton1 or inp.UserInputType==Enum.UserInputType.Touch then
		dragging=true dragStart=inp.Position dragPos=panel.Position end end)
panel.InputEnded:Connect(function(inp)
	if inp.UserInputType==Enum.UserInputType.MouseButton1 or inp.UserInputType==Enum.UserInputType.Touch then
		dragging=false end end)
if UIS then UIS.InputChanged:Connect(function(inp)
	if dragging and (inp.UserInputType==Enum.UserInputType.MouseMovement or inp.UserInputType==Enum.UserInputType.Touch) then
		local d=inp.Position-dragStart
		panel.Position=UDim2.new(dragPos.X.Scale,dragPos.X.Offset+d.X,dragPos.Y.Scale,dragPos.Y.Offset+d.Y) end end) end

local header=Instance.new("Frame")
header.Size=UDim2.new(1,0,0,48) header.BackgroundColor3=Color3.fromRGB(20,20,32)
header.BorderSizePixel=0 header.Parent=panel
Instance.new("UICorner",header).CornerRadius=UDim.new(0,14)

local roleBadge=Instance.new("TextLabel")
roleBadge.Size=UDim2.new(0,60,0,22) roleBadge.Position=UDim2.new(0,12,0,4)
roleBadge.BackgroundColor3=IS_MAIN and Color3.fromRGB(40,190,100) or IS_ALT and Color3.fromRGB(50,110,240) or IS_GUARD and Color3.fromRGB(220,80,80) or Color3.fromRGB(60,60,80)
roleBadge.Text=IS_MAIN and "MAIN" or IS_ALT and "ALT" or IS_GUARD and "GUARD" or "???"
roleBadge.TextColor3=Color3.fromRGB(255,255,255) roleBadge.TextSize=12 roleBadge.Font=Enum.Font.GothamBold
roleBadge.Parent=header Instance.new("UICorner",roleBadge).CornerRadius=UDim.new(0,6)

local titleLbl=Instance.new("TextLabel")
titleLbl.Size=UDim2.new(1,-50,0,20) titleLbl.Position=UDim2.new(0,80,0,4)
titleLbl.BackgroundTransparency=1 titleLbl.Text="⚡ WW Hub v6.2"
titleLbl.TextColor3=Color3.fromRGB(155,80,255) titleLbl.TextSize=17 titleLbl.Font=Enum.Font.GothamBold
titleLbl.TextXAlignment=Enum.TextXAlignment.Left titleLbl.Parent=header

local playerLbl=Instance.new("TextLabel")
playerLbl.Size=UDim2.new(1,-10,0,16) playerLbl.Position=UDim2.new(0,12,0,26)
playerLbl.BackgroundTransparency=1 playerLbl.Text="👤 "..LP.Name
playerLbl.TextColor3=Color3.fromRGB(160,160,180) playerLbl.TextSize=12 playerLbl.Font=Enum.Font.Gotham
playerLbl.TextXAlignment=Enum.TextXAlignment.Left playerLbl.Parent=header

local closeBtn=Instance.new("TextButton")
closeBtn.Size=UDim2.new(0,32,0,32) closeBtn.Position=UDim2.new(1,-40,0,8)
closeBtn.BackgroundColor3=Color3.fromRGB(190,35,55) closeBtn.Text="✕"
closeBtn.TextColor3=Color3.fromRGB(255,255,255) closeBtn.TextSize=15 closeBtn.Font=Enum.Font.GothamBold
closeBtn.Parent=header Instance.new("UICorner",closeBtn).CornerRadius=UDim.new(0,8)

local statusLbl=Instance.new("TextLabel")
statusLbl.Size=UDim2.new(1,-20,0,20) statusLbl.Position=UDim2.new(0,10,0,54)
statusLbl.BackgroundTransparency=1 statusLbl.Text="📊 Idle"
statusLbl.TextColor3=Color3.fromRGB(140,140,165) statusLbl.TextSize=12 statusLbl.Font=Enum.Font.Gotham
statusLbl.TextXAlignment=Enum.TextXAlignment.Left statusLbl.Parent=panel

local div=Instance.new("Frame")
div.Size=UDim2.new(1,-20,0,1) div.Position=UDim2.new(0,10,0,78)
div.BackgroundColor3=Color3.fromRGB(60,35,110) div.BorderSizePixel=0 div.Parent=panel

local function mkBtn(txt,col,y,w,xOff)
	local b=Instance.new("TextButton")
	b.Size=UDim2.new(w or 1,-20,0,48) b.Position=UDim2.new(0,xOff or 10,0,y)
	b.BackgroundColor3=col b.Text=txt b.TextColor3=Color3.fromRGB(255,255,255)
	b.TextSize=18 b.Font=Enum.Font.GothamBold b.Parent=panel
	Instance.new("UICorner",b).CornerRadius=UDim.new(0,10) return b
end

local altBtn   = mkBtn("💀 ALT",   Color3.fromRGB(50,110,240), 88,  0.47, 10)
local mainBtn  = mkBtn("🎮 MAIN",  Color3.fromRGB(50,185,90),  88,  0.47, nil)
mainBtn.Position=UDim2.new(0.53,-8,0,88)
local guardBtn = mkBtn("🛡 GUARD", Color3.fromRGB(200,70,70),  144, 1, 10)
local stopBtn  = mkBtn("⏹ STOP",  Color3.fromRGB(80,80,100),  200, 1, 10)

local capLbl=Instance.new("TextLabel")
capLbl.Size=UDim2.new(1,-20,0,18) capLbl.Position=UDim2.new(0,10,0,256)
capLbl.BackgroundTransparency=1 capLbl.Text="🎯 Cap: "..pointCapLimit
capLbl.TextColor3=Color3.fromRGB(255,200,60) capLbl.TextSize=12 capLbl.Font=Enum.Font.GothamBold
capLbl.TextXAlignment=Enum.TextXAlignment.Left capLbl.Parent=panel

local function mkSBtn(txt,xOff)
	local b=Instance.new("TextButton")
	b.Size=UDim2.new(0,34,0,26) b.Position=UDim2.new(1,xOff,0,252)
	b.BackgroundColor3=Color3.fromRGB(40,40,55) b.Text=txt
	b.TextColor3=Color3.fromRGB(255,255,255) b.TextSize=16 b.Font=Enum.Font.GothamBold
	b.Parent=panel Instance.new("UICorner",b).CornerRadius=UDim.new(0,7) return b
end
local capMinus=mkSBtn("−",-86) local capPlus=mkSBtn("+",-48)

local renderEnabled=true
local renderBtn=mkBtn("👁 Render: ON",Color3.fromRGB(0,120,210),286,1,10)
renderBtn.TextSize=14

local destroyBtn=Instance.new("TextButton")
destroyBtn.Size=UDim2.new(1,-20,0,26) destroyBtn.Position=UDim2.new(0,10,1,-34)
destroyBtn.BackgroundColor3=Color3.fromRGB(35,35,50) destroyBtn.Text="❌ Destroy GUI"
destroyBtn.TextColor3=Color3.fromRGB(200,200,220) destroyBtn.TextSize=12 destroyBtn.Font=Enum.Font.GothamBold
destroyBtn.Parent=panel Instance.new("UICorner",destroyBtn).CornerRadius=UDim.new(0,8)

local function setStatus(txt) statusLbl.Text="📊 "..txt end

toggleBtn.MouseButton1Click:Connect(function() panel.Visible=not panel.Visible end)
altBtn.MouseButton1Click:Connect(function()
	task.spawn(function() startFarm("alt") end)
	altBtn.BackgroundColor3=Color3.fromRGB(40,190,100) altBtn.Text="⏸ ALT"
	mainBtn.BackgroundColor3=Color3.fromRGB(50,185,90) mainBtn.Text="🎮 MAIN"
	guardBtn.BackgroundColor3=Color3.fromRGB(200,70,70) guardBtn.Text="🛡 GUARD"
end)
mainBtn.MouseButton1Click:Connect(function()
	task.spawn(function() startFarm("main") end)
	mainBtn.BackgroundColor3=Color3.fromRGB(40,190,100) mainBtn.Text="⏸ MAIN"
	altBtn.BackgroundColor3=Color3.fromRGB(50,110,240) altBtn.Text="💀 ALT"
	guardBtn.BackgroundColor3=Color3.fromRGB(200,70,70) guardBtn.Text="🛡 GUARD"
end)
guardBtn.MouseButton1Click:Connect(function()
	task.spawn(function() startFarm("guard") end)
	guardBtn.BackgroundColor3=Color3.fromRGB(240,80,80) guardBtn.Text="⏸ GUARD"
	altBtn.BackgroundColor3=Color3.fromRGB(50,110,240) altBtn.Text="💀 ALT"
	mainBtn.BackgroundColor3=Color3.fromRGB(50,185,90) mainBtn.Text="🎮 MAIN"
end)
stopBtn.MouseButton1Click:Connect(function()
	loopAlt=false loopMain=false loopGuard=false
	altBtn.BackgroundColor3=Color3.fromRGB(50,110,240) altBtn.Text="💀 ALT"
	mainBtn.BackgroundColor3=Color3.fromRGB(50,185,90) mainBtn.Text="🎮 MAIN"
	guardBtn.BackgroundColor3=Color3.fromRGB(200,70,70) guardBtn.Text="🛡 GUARD"
	setStatus("Idle")
end)
capMinus.MouseButton1Click:Connect(function()
	pointCapLimit=math.max(10000,pointCapLimit-10000) capLbl.Text="🎯 Cap: "..pointCapLimit end)
capPlus.MouseButton1Click:Connect(function()
	pointCapLimit=pointCapLimit+10000 capLbl.Text="🎯 Cap: "..pointCapLimit end)
renderBtn.MouseButton1Click:Connect(function()
	renderEnabled=not renderEnabled
	game:GetService("RunService"):Set3dRenderingEnabled(renderEnabled)
	renderBtn.Text=renderEnabled and "👁 Render: ON" or "👁 Render: OFF"
	renderBtn.BackgroundColor3=renderEnabled and Color3.fromRGB(0,120,210) or Color3.fromRGB(170,35,35)
end)
closeBtn.MouseButton1Click:Connect(function() panel.Visible=false end)
destroyBtn.MouseButton1Click:Connect(function() loopAlt=false loopMain=false loopGuard=false gui:Destroy() end)

-- Auto-start
task.spawn(function()
	task.wait(0.5)
	if IS_MAIN then
		task.spawn(function() startFarm("main") end)
		mainBtn.BackgroundColor3=Color3.fromRGB(40,190,100) mainBtn.Text="⏸ MAIN"
	elseif IS_ALT then
		task.spawn(function() startFarm("alt") end)
		altBtn.BackgroundColor3=Color3.fromRGB(40,190,100) altBtn.Text="⏸ ALT"
	elseif IS_GUARD then
		task.spawn(function() startFarm("guard") end)
		guardBtn.BackgroundColor3=Color3.fromRGB(240,80,80) guardBtn.Text="⏸ GUARD"
	end
end)

-- NoClip main
local noClipOn=false
task.spawn(function()
	while gui.Parent do
		if loopMain and not roundPaused and not timerTpDone then
			if not noClipOn then noClipOn=true end
			local c=getChar()
			if c then
				for _,p in ipairs(c:GetDescendants()) do if p:IsA("BasePart") then p.CanCollide=false end end
				local hrp=getHRP()
				if hrp and hrp.Position.Y<-10 then
					pcall(function() hrp.CFrame=CFrame.new(hrp.Position.X,5,hrp.Position.Z) hrp.AssemblyLinearVelocity=Vector3.new(0,0,0) end)
				end
			end
		else
			if noClipOn then
				noClipOn=false local c=getChar()
				if c then for _,p in ipairs(c:GetDescendants()) do if p:IsA("BasePart") then p.CanCollide=true end end end
			end
		end
		task.wait(0.1)
	end
end)

-- Alt wrong team
task.spawn(function()
	while gui.Parent do
		task.wait(1)
		if not loopAlt or roundPaused or timerTpDone then continue end
		if not hasTeamPads() then continue end
		local ok,onRed=pcall(function() local t=LP.Team return t~=nil and t.Name:lower():find("red")~=nil end)
		if not (ok and onRed) then continue end
		task.wait(1)
		if not loopAlt or roundPaused or timerTpDone then continue end
		if not hasTeamPads() then continue end
		local ok2,stillRed=pcall(function() local t=LP.Team return t~=nil and t.Name:lower():find("red")~=nil end)
		if not (ok2 and stillRed) then continue end
		warn("[WWHub] Alt on Red — pausing")
		roundPaused=true roundPauseReason="Wrong Team"
		tpToSafeZone()
		local waited=0
		repeat task.wait(1) waited+=1
			if not hasTeamPads() then break end
			if workspace:FindFirstChild("Blue Team") then break end
		until waited>=90 or not gui.Parent or not loopAlt
		task.wait(1.5) roundPaused=false roundPauseReason=nil
	end
end)

-- Status sync
task.spawn(function()
	while gui and gui.Parent do
		local altCount=countAlts()
		local info=" | alt="..altCount.."/"..#_G.alt.." | guard="..countGuards().."/"..#_G.guard.." | Lv:"..getLevel()
		local hpInfo=""
		if loopAlt then
			local hum=getHum()
			if hum and hum.MaxHealth>0 then hpInfo=" | HP:"..math.floor(hum.Health/hum.MaxHealth*100).."%" end
		end
		if starting then
			statusLbl.TextColor3=Color3.fromRGB(255,200,50) statusLbl.Font=Enum.Font.GothamBold statusLbl.TextSize=12
			setStatus("Starting...")
		elseif roundPaused then
			statusLbl.TextColor3=Color3.fromRGB(255,80,80) statusLbl.Font=Enum.Font.GothamBold statusLbl.TextSize=12
			setStatus("⏸ "..(roundPauseReason or "Paused")..info)
		elseif timerTpDone then
			statusLbl.TextColor3=Color3.fromRGB(255,165,0) statusLbl.Font=Enum.Font.GothamBold statusLbl.TextSize=12
			setStatus("⏱ Safe zone"..info)
		elseif pointsCapped then
			statusLbl.TextColor3=Color3.fromRGB(255,215,0) statusLbl.Font=Enum.Font.GothamBold statusLbl.TextSize=12
			setStatus("🎯 Cap! pts="..getPoints()..info)
		elseif loopGuard then
			statusLbl.TextColor3=Color3.fromRGB(255,100,100) statusLbl.Font=Enum.Font.GothamBold statusLbl.TextSize=13
			setStatus("🛡 GUARD | targets="..#getTargets()..info)
		elseif loopAlt then
			statusLbl.TextColor3=Color3.fromRGB(80,255,140) statusLbl.Font=Enum.Font.GothamBold statusLbl.TextSize=13
			setStatus("💀 ALT | t="..getTimerValue()..info..hpInfo)
		elseif loopMain then
			statusLbl.TextColor3=Color3.fromRGB(100,200,255) statusLbl.Font=Enum.Font.GothamBold statusLbl.TextSize=13
			setStatus("🎮 MAIN | t="..getTimerValue()..info)
		else
			statusLbl.TextColor3=Color3.fromRGB(140,140,165) statusLbl.Font=Enum.Font.Gotham statusLbl.TextSize=12
			setStatus("Idle"..info)
		end
		task.wait(0.5)
	end
end)

-- ===== Farm Loops =====
task.spawn(function()
	makeBase()
	while gui.Parent do
		local c=LP.Character
		if c and c.Parent and c~=handledChar then afterCharLoaded(c) end
		if not roundPaused and not timerTpDone then pressKAfterTP() end
		task.wait(0.25)
	end
end)

-- TP loop (main/alt เท่านั้น guard จัดการ TP เอง)
task.spawn(function()
	while gui.Parent do
		if not (loopAlt or loopMain) then task.wait(0.5) continue end
		local pad=getTeamPad()
		if pad and not selectingTeam and not roundPaused and not timerTpDone then task.spawn(selectTeam) end
		if not selectingTeam and not starting and not roundPaused and not timerTpDone then
			if loopAlt and not isNearCF(altCFrame,tpDistLimit) then tpAndVerify(altCFrame)
			elseif loopMain then
				local mcf=getMainCF()
				if not isNearCF(mcf,tpDistLimit) then tpAndVerify(mcf) end
			end
		end
		task.wait(0.08)
	end
end)

task.spawn(function()
	while gui.Parent do
		if loopMain and not starting and not selectingTeam and not roundPaused and not pointsCapped and not timerTpDone then fireM1() end
		task.wait(0.12)
	end
end)

task.spawn(function()
	while gui.Parent do
		if loopMain and not starting and not selectingTeam and not roundPaused and not pointsCapped and not timerTpDone then fireSkills() end
		task.wait(0.8)
	end
end)

task.spawn(function()
	while gui.Parent do
		if (loopMain or loopAlt) and not starting and not roundPaused then
			local bm=getBlockedMode() if bm then pauseFarm(bm) end task.wait(0.5)
		else task.wait(2) end
	end
end)

task.spawn(function()
	local modes={"Kills Team","Team Battle","3 Teams","Free For All","Kills FFA"}
	while gui.Parent do
		if (loopAlt or loopMain) and not roundPaused then
			for _,m in ipairs(modes) do fireInput("mode",m) task.wait(0.3) end task.wait(2)
		else task.wait(3) end
	end
end)

-- ===== GUARD Loop =====
task.spawn(function()
	while gui.Parent do
		if not loopGuard then task.wait(1) continue end

		local targets = getTargets()

		-- ไม่มีคนนอก → ทำหน้าที่เหมือน alt (TP ไป altCFrame ให้ main ตี)
		if #targets == 0 then
			local bluePad = workspace:FindFirstChild("Blue Team")
			if bluePad and not selectingTeam then
				task.spawn(function()
					if selectingTeam then return end
					selectingTeam = true
					while workspace:FindFirstChild("Blue Team") and #getTargets() == 0 do
						local hrp = getHRP()
						if hrp then
							local pp = bluePad:IsA("BasePart") and bluePad or bluePad:FindFirstChildWhichIsA("BasePart")
							if pp then hrp.CFrame = pp.CFrame + Vector3.new(0,3,0) end
						end
						task.wait(0.1)
					end
					task.wait(0.5)
					selectingTeam = false
				end)
			end
			-- TP ไป altCFrame โดยตรง ไม่ชนกับ alt TP loop
			if not selectingTeam and not isNearCF(altCFrame, tpDistLimit) then
				tpToCF(altCFrame)
			end
			task.wait(0.08)
			continue
		end

		-- มีคนนอก → guard โจมตีทีละคน 5 วิ
		for _, target in ipairs(targets) do
			if not loopGuard or not gui.Parent then break end
			-- ถ้า target หายระหว่างรอบ break ออกเช็คใหม่
			if #getTargets() == 0 then break end

			local tChar = target.Character
			if not tChar or not tChar.Parent then continue end
			local tHRP = tChar:FindFirstChild("HumanoidRootPart")
			if not tHRP or not tHRP.Parent then continue end

			local attackEnd = tick() + 5
			while tick() < attackEnd and loopGuard and gui.Parent do
				-- ถ้าระหว่างตีคนนอกหายหมด ออกไปเป็น alt
				if #getTargets() == 0 then break end

				tChar = target.Character
				if not tChar or not tChar.Parent then break end
				tHRP = tChar:FindFirstChild("HumanoidRootPart")
				if not tHRP or not tHRP.Parent then break end

				-- TP ติดตาม target
				local hrp = getHRP()
				if hrp then
					pcall(function()
						hrp.AssemblyLinearVelocity = Vector3.new(0,0,0)
						hrp.AssemblyAngularVelocity = Vector3.new(0,0,0)
						hrp.CFrame = tHRP.CFrame * CFrame.new(0, 0, 2.5)
					end)
				end

				fireM1()
				task.wait(0.1)
				pressG()
				task.wait(0.08)
			end

			if loopGuard then fireSkills() end
			task.wait(0.3)
		end
	end
end)

task.spawn(function()
	while gui.Parent do task.wait(30) if loopMain and not roundPaused then sendWebhook("Main Farm Report") end end
end)
task.spawn(function()
	while gui and gui.Parent do task.wait(60) collectgarbage("collect") end
end)

-- Rejoin เฉพาะ main เมื่อ alt หายหมด (hop server ใหม่)
-- ไม่มี rejoin จาก kick หรือ heartbeat แล้ว
