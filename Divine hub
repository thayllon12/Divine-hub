--// SERVIÇOS
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local VirtualInputManager = game:GetService("VirtualInputManager")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")
local Camera = workspace.CurrentCamera

--// REMOVER GUI ANTIGA
if PlayerGui:FindFirstChild("DivineHub") then
	PlayerGui.DivineHub:Destroy()
end

--// VARIÁVEIS
local SelectedPlayer = nil
local TweenActive = false
local SilentActive = false
local LOCKED = false
local aimbot_target = nil
local last_target = nil
local SPEED = 350
local UP_OFFSET = 30
local ORBIT_RADIUS = 15
local ORBIT_HEIGHT = 15
local ORBIT_SPEED = 5

local AlturaSubida = nil
local Subiu = false
--============================================================
-- GUI BASE
--============================================================
local Gui = Instance.new("ScreenGui", PlayerGui)
Gui.Name = "DivineHub"
Gui.ResetOnSpawn = false

-- BOTÃO PRINCIPAL DIVINE HUB (30x30, RGB, ARRASTÁVEL, TOGGLE MENU)
local mainButton = Instance.new("ImageButton", Gui) -- Gui é seu DivineHub ScreenGui
mainButton.Size = UDim2.new(0,40,0,40)
mainButton.Position = UDim2.new(0,15,0,80)
mainButton.BackgroundColor3 = Color3.fromRGB(0, 255, 255)
mainButton.Image = "rbxassetid://75330746621779"
mainButton.ImageColor3 = Color3.new(1,1,1)
mainButton.AutoButtonColor = true
Instance.new("UICorner", mainButton).CornerRadius = UDim.new(0,14)
local stroke = Instance.new("UIStroke", mainButton)
stroke.Thickness = 3

-- RGB + PULSAR
task.spawn(function()
    local h = 0
    while true do
        h = (h + 0.01) % 1
        stroke.Color = Color3.fromHSV(h,1,1)
        mainButton.BackgroundColor3 = Color3.fromHSV((tick()*0.3)%1,1,1)
        mainButton.Size = UDim2.new(0,40 + math.sin(tick()*3)*4,0,40 + math.sin(tick()*3)*4)
        task.wait(0.03)
    end
end)

-- SISTEMA DE ARRASTAR
local draggingMain = false
local dragInputMain, startPosMain, startMousePosMain

local function updateMain(input)
    local delta = input.Position - startMousePosMain
    mainButton.Position = UDim2.new(
        startPosMain.X.Scale,
        startPosMain.X.Offset + delta.X,
        startPosMain.Y.Scale,
        startPosMain.Y.Offset + delta.Y
    )
end

mainButton.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        draggingMain = true
        startMousePosMain = input.Position
        startPosMain = mainButton.Position

  input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                draggingMain = false
            end
        end)
    end
end)

mainButton.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        dragInputMain = input
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if input == dragInputMain and draggingMain then
        updateMain(input)
    end
end)


-- MENU
local MenuFrame = Instance.new("Frame", Gui)
MenuFrame.Size = UDim2.new(0,420,0,300)
MenuFrame.Position = UDim2.new(0.5,-210,0.5,-150)
MenuFrame.BackgroundColor3 = Color3.fromRGB(20,20,20)
MenuFrame.Visible = false
Instance.new("UICorner", MenuFrame).CornerRadius = UDim.new(0,14)

-- TOGGLE MENU
mainButton.MouseButton1Click:Connect(function()
    MenuFrame.Visible = not MenuFrame.Visible
end)

--============================================================
-- BOTÕES
--============================================================
local function AddBtn(parent,text,pos)
	local b = Instance.new("TextButton", parent)
	b.Size = UDim2.new(0,120,0,45)
	b.Position = pos
	b.BackgroundColor3 = Color3.fromRGB(120,10,10)
	b.Text = text
	b.TextColor3 = Color3.new(1,1,1)
	b.Font = Enum.Font.GothamBold
	b.TextSize = 15
	Instance.new("UICorner", b).CornerRadius = UDim.new(0,10)
	return b
end

local TweenBtn = AddBtn(MenuFrame,"Tween OFF",UDim2.new(0,15,0,20))
local SilentBtn = AddBtn(MenuFrame,"Silent OFF",UDim2.new(0,150,0,20))
local KillBtn = AddBtn(MenuFrame,"Kill OFF",UDim2.new(0,285,0,20))
local FastBtn = AddBtn(MenuFrame,"Fast OFF",UDim2.new(0,15,0,140))
local SelectBtn = AddBtn(MenuFrame,"Selecionar",UDim2.new(0,15,0,80))
local RemoveBtn = AddBtn(MenuFrame,"Remover",UDim2.new(0,150,0,80))
local MiscBtn = AddBtn(MenuFrame, "Misc", UDim2.new(0,285,0,80))

-- LABEL ALVO
local SelectedLabel = Instance.new("TextLabel", MenuFrame)
SelectedLabel.Size = UDim2.new(1,0,0,30)
SelectedLabel.Position = UDim2.new(0,0,0,250)
SelectedLabel.BackgroundTransparency = 1
SelectedLabel.Text = "Alvo: Nenhum"
SelectedLabel.TextColor3 = Color3.new(1,255,1)
SelectedLabel.Font = Enum.Font.GothamBold
SelectedLabel.TextSize = 16

-- HUD-SP
local HUD = Instance.new("ScreenGui", PlayerGui)
HUD.Name = "HUD_SP"
HUD.ResetOnSpawn = false

local frame = Instance.new("Frame", HUD)
frame.Size = UDim2.new(0,150,0,50)
frame.Position = UDim2.new(0.5, -75, 0, 50)
frame.BackgroundColor3 = Color3.fromRGB(0,0,0)
frame.BackgroundTransparency = 0.4
frame.BorderSizePixel = 0
Instance.new("UICorner", frame).CornerRadius = UDim.new(0,8)

local txt = Instance.new("TextLabel", frame)
txt.Size = UDim2.new(1,0,1,0)
txt.BackgroundTransparency = 1
txt.TextColor3 = Color3.fromRGB(255,255,255)
txt.Font = Enum.Font.GothamBold
txt.TextSize = 16
txt.Text = "Seleciona alguém"

-- Atualização do HUD
RunService.RenderStepped:Connect(function()
    if SelectedPlayer and SelectedPlayer.Character and SelectedPlayer.Character:FindFirstChild("Humanoid") and SelectedPlayer.Character:FindFirstChild("HumanoidRootPart") then
        local hum = SelectedPlayer.Character:FindFirstChild("Humanoid")
        local root = SelectedPlayer.Character:FindFirstChild("HumanoidRootPart")
        local dist = (root.Position - LocalPlayer.Character.HumanoidRootPart.Position).Magnitude
        txt.Text = string.format("%s\nDistância: %.1f\nVida: %.0f", SelectedPlayer.Name, dist, hum.Health)
        frame.Visible = true
    else
        txt.Text = "Seleciona alguém"
        frame.Visible = false
    end
end)

--============================================================
-- LISTA DE JOGADORES
--============================================================
local PlayerListFrame = Instance.new("Frame", Gui)
PlayerListFrame.Size = UDim2.new(0,210,0,320)
PlayerListFrame.Position = UDim2.new(0.5,140,0.5,-160)
PlayerListFrame.BackgroundColor3 = Color3.fromRGB(20,20,20)
PlayerListFrame.Visible = false
Instance.new("UICorner", PlayerListFrame)

local Scrolling = Instance.new("ScrollingFrame", PlayerListFrame)
Scrolling.Size = UDim2.new(1,0,1,0)
Scrolling.CanvasSize = UDim2.new(0,0,0,0)
Scrolling.ScrollBarThickness = 6

local UIList = Instance.new("UIListLayout", Scrolling)
UIList.Padding = UDim.new(0,6)

local function UpdateList()
	for _,v in pairs(Scrolling:GetChildren()) do
		if v:IsA("TextButton") then v:Destroy() end
	end
	for _,plr in pairs(Players:GetPlayers()) do
		if plr ~= LocalPlayer then
			local b = Instance.new("TextButton", Scrolling)
			b.Size = UDim2.new(1,-10,0,35)
			b.BackgroundColor3 = Color3.fromRGB(50,50,50)
			b.Text = plr.DisplayName
			b.TextColor3 = Color3.new(1,1,1)
			b.Font = Enum.Font.GothamBold
			Instance.new("UICorner", b)
	b.MouseButton1Click:Connect(function()
				SelectedPlayer = plr
				SelectedLabel.Text = "Alvo: "..plr.DisplayName
				PlayerListFrame.Visible = false
			end)
		end
	end

task.wait()
	Scrolling.CanvasSize = UDim2.new(0,0,0,UIList.AbsoluteContentSize.Y+10)
end

SelectBtn.MouseButton1Click:Connect(function()
	PlayerListFrame.Visible = not PlayerListFrame.Visible
	UpdateList()
end)

RemoveBtn.MouseButton1Click:Connect(function()
	SelectedPlayer = nil
	SelectedLabel.Text = "Alvo: Nenhum"
end)

--============================================================
-- FUNÇÕES BASE
--============================================================
-- FUNÇÃO GET PART
local function GetPart(char)
    return char:FindFirstChild("HumanoidRootPart") or char:FindFirstChild("Torso") or char:FindFirstChild("UpperTorso")
end

-- VARIÁVEIS DE NOCOLIP
local noclipConnection = nil
local originalCollisionStates = {}

local function Noclip(state)
    local char = LocalPlayer.Character
    if not char then return end
    if state then
        for _,v in pairs(char:GetDescendants()) do
            if v:IsA("BasePart") then
                originalCollisionStates[v] = v.CanCollide
                v.CanCollide = false
            end
        end
    else
        for part,collide in pairs(originalCollisionStates) do
            if part and part.Parent then
                part.CanCollide = collide
            end
        end
        originalCollisionStates = {}
    end
end

--============================================================
-- TWEEN (O QUE VOCÊ MANDOU)
--============================================================

-- TWEEN BOTÃO
TweenBtn.MouseButton1Click:Connect(function()
	if not SelectedPlayer then return end
	TweenActive = not TweenActive
	TweenBtn.Text = TweenActive and "Tween ON" or "Tween OFF"
	TweenBtn.BackgroundColor3 = TweenActive and Color3.fromRGB(20,120,20) or Color3.fromRGB(120,10,10)

if TweenActive then
		local hrp = GetPart(LocalPlayer.Character)
		if hrp then
			AlturaSubida = hrp.Position.Y + UP_OFFSET
		end
		Subiu = false
		Noclip(true)
	else
		Subiu = false
		Noclip(false)
	end
end)

-- LOOP DO TWEEN
RunService.Heartbeat:Connect(function(dt)
	if not TweenActive then return end
	if not SelectedPlayer or not SelectedPlayer.Character or not SelectedPlayer.Character:FindFirstChildOfClass("Humanoid") 
		or SelectedPlayer.Character.Humanoid.Health <= 0 then
		Subiu = false
		for _,plr in pairs(Players:GetPlayers()) do
			if plr ~= LocalPlayer then
				SelectedPlayer = plr
				break
			end
		end
		return
	end

local hrp = GetPart(LocalPlayer.Character)
	local targetHRP = GetPart(SelectedPlayer.Character)
	if not hrp or not targetHRP then return end
	-- SUBIDA INICIAL
	if not Subiu and AlturaSubida then
		local destino = Vector3.new(hrp.Position.X, AlturaSubida, hrp.Position.Z)
		local move = destino - hrp.Position
		if move.Magnitude < 1 then
			Subiu = true
		else
			hrp.CFrame = hrp.CFrame + move.Unit * math.min(move.Magnitude, SPEED * dt)
		end
		return
	end

-- ORBITAL
	local angle = tick() * ORBIT_SPEED
	local offset = Vector3.new(math.cos(angle)*ORBIT_RADIUS, ORBIT_HEIGHT, math.sin(angle)*ORBIT_RADIUS)
	local destination = targetHRP.Position + offset
	local move = destination - hrp.Position
	if move.Magnitude > 0.1 then
		hrp.CFrame = CFrame.new(hrp.Position + move.Unit * math.min(move.Magnitude, SPEED * dt), targetHRP.Position)
	end
	hrp.Velocity = Vector3.zero
end)

--============================================================
-- SILENT AIM BOTÃO
--============================================================
SilentBtn.MouseButton1Click:Connect(function()
	SilentActive = not SilentActive
	SilentBtn.Text = SilentActive and "Silent ON" or "Silent OFF"
	SilentBtn.BackgroundColor3 = SilentActive and Color3.fromRGB(20,120,20) or Color3.fromRGB(120,10,10)
end)

--============================================================
-- SILENT AIM GUI + CADEADO
--============================================================
local SilentGui, Profile, ImageProfile, NamePlayers, HealthPlayers, Healthbar, Healthgreen, LockBtn

local function GetClosestForSilent()
	local myPos = LocalPlayer.Character and GetPart(LocalPlayer.Character).Position
	if not myPos then return nil end
	local closest, best = nil, math.huge
	for _,p in pairs(Players:GetPlayers()) do
		if p ~= LocalPlayer and p.Character and GetPart(p.Character) then
			local hum = p.Character:FindFirstChildOfClass("Humanoid")
			if hum and hum.Health > 0 then
				local dist = (GetPart(p.Character).Position - myPos).Magnitude
				if dist < best then
					best = dist
					closest = p
				end
			end
		end
	end
	return closest
end

local function CreateSilentGui()
	if SilentGui then SilentGui:Destroy() end

SilentGui = Instance.new("Frame", Gui)
	SilentGui.Size = UDim2.new(0,220,0,95)
	SilentGui.Position = UDim2.new(0.03,0,0.4,0)
	SilentGui.BackgroundColor3 = Color3.fromRGB(40,40,40)
	SilentGui.Active = true
	SilentGui.Draggable = true
	Instance.new("UICorner", SilentGui)
	Profile = Instance.new("Frame", SilentGui)
	Profile.Size = UDim2.new(0,50,0,50)
	Profile.Position = UDim2.new(0.02,0,0.1,0)
	Profile.BackgroundColor3 = Color3.fromRGB(255,255,255)
	Instance.new("UICorner", Profile)

ImageProfile = Instance.new("ImageLabel", Profile)
	ImageProfile.Size = UDim2.new(0,48,0,48)
	ImageProfile.Position = UDim2.new(0,1,0,1)
	Instance.new("UICorner", ImageProfile)
	NamePlayers = Instance.new("TextLabel", Profile)
	NamePlayers.Position = UDim2.new(1.3,0,0.05,0)
	NamePlayers.Size = UDim2.new(0,140,0,20)
	NamePlayers.BackgroundTransparency = 1
	NamePlayers.TextColor3 = Color3.new(1,1,1)
	NamePlayers.Font = Enum.Font.GothamBold
	NamePlayers.TextSize = 16

  HealthPlayers = Instance.new("TextLabel", Profile)
	HealthPlayers.Position = UDim2.new(1.3,0,0.45,0)
	HealthPlayers.Size = UDim2.new(0,140,0,18)
	HealthPlayers.BackgroundTransparency = 1
	HealthPlayers.TextColor3 = Color3.new(1,1,1)

  Healthbar = Instance.new("Frame", Profile)
	Healthbar.Position = UDim2.new(1.3,0,0.8,0)
	Healthbar.Size = UDim2.new(0,130,0,7)
	Healthbar.BackgroundColor3 = Color3.fromRGB(40,40,40)
	Instance.new("UICorner", Healthbar)
	Healthgreen = Instance.new("Frame", Healthbar)
	Healthgreen.Size = UDim2.new(1,0,1,0)
	Healthgreen.BackgroundColor3 = Color3.fromRGB(0,220,100)
	Instance.new("UICorner", Healthgreen)
	LockBtn = Instance.new("TextButton", SilentGui)
	LockBtn.Size = UDim2.new(0,35,0,35)
	LockBtn.Position = UDim2.new(0.75,0,0.85,0)
	LockBtn.Text = "🔓"
	LockBtn.Font = Enum.Font.GothamBold
	LockBtn.TextSize = 20
	LockBtn.MouseButton1Click:Connect(function()
		LOCKED = not LOCKED
		LockBtn.Text = LOCKED and "🔒" or "🔓"
	end)
end

RunService.RenderStepped:Connect(function()
	if not SilentActive then
		if SilentGui then SilentGui:Destroy(); SilentGui=nil end
		aimbot_target = nil
		return
	end
	if not SilentGui then CreateSilentGui() end
	local target = LOCKED and last_target or GetClosestForSilent()
	last_target = target
	if target and target.Character then
		aimbot_target = GetPart(target.Character).Position + Vector3.new(0,3,0)
		NamePlayers.Text = "Name | "..target.DisplayName
		HealthPlayers.Text = "Health | "..math.floor(target.Character.Humanoid.Health)
		Healthgreen.Size = UDim2.new(target.Character.Humanoid.Health/target.Character.Humanoid.MaxHealth,0,1,0)
		ImageProfile.Image = Players:GetUserThumbnailAsync(target.UserId,Enum.ThumbnailType.HeadShot,Enum.ThumbnailSize.Size150x150)
	end
end)

--============================================================
-- HOOK SILENT AIM
--============================================================
local old
old = hookmetamethod(game, "__namecall", function(self,...)
	local args = {...}
	if aimbot_target and (getnamecallmethod()=="FireServer" or getnamecallmethod()=="InvokeServer") then
		if typeof(args[1])=="Vector3" then args[1] = aimbot_target end
		if typeof(args[2])=="Vector3" then args[2] = aimbot_target end
	end
	return old(self, unpack(args))
end) 
--========================================================
-- SISTEMA KILL (BOTÃO DO HUB)
--========================================================

local ORBIT_RADIUS = 15
local ORBIT_HEIGHT = 15
local UP_OFFSET = 30
local MIN_LEVEL = 2100
local ORBIT_SPEED = 5
local ATTACK_DISTANCE = 40

local KillActive = false
local Subiu = false
local AlturaSubida = nil
local AutoSkill = true
local Skillx, Skillz, Skillc = true, true, true

local function GetPlayerLevel(player)
	if player:FindFirstChild("Data") and player.Data:FindFirstChild("Level") then
		return player.Data.Level.Value
	end
	return 0
end

local function GetClosestKillPlayer()
	local char = LocalPlayer.Character
	if not char or not char:FindFirstChild("HumanoidRootPart") then return nil end
	local myPos = char.HumanoidRootPart.Position
	local closest, dist = nil, math.huge
	for _, plr in pairs(Players:GetPlayers()) do
		if plr ~= LocalPlayer and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
			local hum = plr.Character:FindFirstChildOfClass("Humanoid")
			local level = GetPlayerLevel(plr)
			if hum and hum.Health > 0 and level >= MIN_LEVEL then
				local d = (plr.Character.HumanoidRootPart.Position - myPos).Magnitude
				if d < dist then
					dist = d
					closest = plr
				end
			end
		end
	end
	return closest
end

-- BOTÃO KILL
KillBtn.MouseButton1Click:Connect(function()
	KillActive = not KillActive
	KillBtn.Text = KillActive and "Kill ON" or "Kill OFF"
	KillBtn.BackgroundColor3 = KillActive and Color3.fromRGB(20,120,20) or Color3.fromRGB(120,10,10)
	if KillActive then
		SelectedPlayer = GetClosestKillPlayer()
		Subiu = false
		local hrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
		if hrp then AlturaSubida = hrp.Position.Y + UP_OFFSET end
	else
		SelectedPlayer = nil
		AlturaSubida = nil
	end
end)

-- AUTO EQUIP GODHUMAN
task.spawn(function()
	while task.wait(3) do
		if KillActive then
			pcall(function()
				local char = LocalPlayer.Character
				if char then
					local hum = char:FindFirstChildOfClass("Humanoid")
					local tool = LocalPlayer.Backpack:FindFirstChild("Godhuman") or char:FindFirstChild("Godhuman")
					if tool and hum then
						hum:EquipTool(tool)
					end
				end
			end)
		end
	end
end)

-- MOVIMENTO KILL
RunService.Heartbeat:Connect(function(dt)
	if not KillActive then return end
	if not SelectedPlayer or not SelectedPlayer.Character 
		or SelectedPlayer.Character:FindFirstChildOfClass("Humanoid").Health <= 0 
		or GetPlayerLevel(SelectedPlayer) < MIN_LEVEL then
 	SelectedPlayer = GetClosestKillPlayer()
		Subiu = false
		local hrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
		if hrp then AlturaSubida = hrp.Position.Y + UP_OFFSET end
		return
	end
	local char = LocalPlayer.Character
	local hrp = char and char:FindFirstChild("HumanoidRootPart")
	local targetHRP = SelectedPlayer.Character:FindFirstChild("HumanoidRootPart")
	if not hrp or not targetHRP then return end
	if not Subiu and AlturaSubida then
		local destino = Vector3.new(hrp.Position.X, AlturaSubida, hrp.Position.Z)
		local move = destino - hrp.Position
		local dist = move.Magnitude
		if dist > 1 then
			hrp.CFrame = hrp.CFrame + move.Unit * math.min(dist, SPEED * dt)
		else
			Subiu = true
		end
		return
	end
	local angle = tick() * ORBIT_SPEED
	local offset = Vector3.new(
		math.cos(angle) * ORBIT_RADIUS,
		ORBIT_HEIGHT,
		math.sin(angle) * ORBIT_RADIUS
	)
	local destination = targetHRP.Position + offset
  local move = destination - hrp.Position
	local dist = move.Magnitude

if dist > 0.1 then
		hrp.CFrame = CFrame.new(hrp.Position + move.Unit * math.min(dist, SPEED * dt), targetHRP.Position)
	end
	hrp.Velocity = Vector3.zero
end)

-- AUTO SKILL X Z C
task.spawn(function()
	while task.wait() do
		if AutoSkill and KillActive and SelectedPlayer and SelectedPlayer.Character then
			local hrp = LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
			local targetHRP = SelectedPlayer.Character:FindFirstChild("HumanoidRootPart")
			if hrp and targetHRP then
				local distance = (hrp.Position - targetHRP.Position).Magnitude
				if distance <= ATTACK_DISTANCE then
					if Skillx then
						VirtualInputManager:SendKeyEvent(true, "X", false, game)
						task.wait(0.1)
						VirtualInputManager:SendKeyEvent(false, "X", false, game)
					end
					if Skillz then
						VirtualInputManager:SendKeyEvent(true, "Z", false, game)
						task.wait(0.1)
						VirtualInputManager:SendKeyEvent(false, "Z", false, game)
					end
					if Skillc then
						VirtualInputManager:SendKeyEvent(true, "C", false, game)
						task.wait(0.1)
						VirtualInputManager:SendKeyEvent(false, "C", false, game)
					end
				end
			end
		end
	end
end)
--========================================================
-- FAST ATTACK (BOTÃO DO HUB)
--========================================================

--===============================
-- FAST ATTACK + FRUTAS | Divine
--===============================

local FastAttackActive = false
local FRUIT_ACTIVE = false
local MELEE_SPEED = 0.002

local Modules = ReplicatedStorage:WaitForChild("Modules")
local Net = Modules:WaitForChild("Net")
local RegisterAttack = Net:WaitForChild("RE/RegisterAttack")
local RegisterHit = Net:WaitForChild("RE/RegisterHit")

local Fruits = {
	"Kitsune-Kitsune","Pain-Pain","Blade-Blade",
	"Mammoth-Mammoth","Gas-Gas","Tiger-Tiger",
	"Yeti-Yeti","Dragon-Dragon"
}

local function GetPart(m)
	return m and (m:FindFirstChild("HumanoidRootPart") or m:FindFirstChild("Torso"))
end

local function GetClosestFastTarget(range)
	local hrp = GetPart(LocalPlayer.Character)
	if not hrp then return nil end

local closest, dist = nil, range
	-- PLAYERS
	for _, plr in ipairs(Players:GetPlayers()) do
		if plr ~= LocalPlayer and plr.Character then
			local root = GetPart(plr.Character)
			local hum = plr.Character:FindFirstChildOfClass("Humanoid")
			if root and hum and hum.Health > 0 then
				local d = (root.Position - hrp.Position).Magnitude
				if d < dist then
					dist = d
					closest = plr.Character
				end
			end
		end
	end

-- NPCs
	if workspace:FindFirstChild("Enemies") then
		for _, npc in ipairs(workspace.Enemies:GetChildren()) do
			local root = GetPart(npc)
			local hum = npc:FindFirstChildOfClass("Humanoid")
			if root and hum and hum.Health > 0 then
				local d = (root.Position - hrp.Position).Magnitude
				if d < dist then
					dist = d
					closest = npc
				end
			end
		end
	end
	return closest
end

-- LOOP PRINCIPAL FAST ATTACK + FRUTAS
task.spawn(function()
	while task.wait(0.01) do
		if FastAttackActive then
			local target = GetClosestFastTarget(9999)
			if target then
				-- FAST MELEE
				pcall(function()
					local p = GetPart(target)
					if p then
						RegisterAttack:FireServer(MELEE_SPEED)
						RegisterHit:FireServer(p, {}, nil, "24057351")
					end
				end)

-- FAST FRUIT
				if FRUIT_ACTIVE then
					pcall(function()
						local p = GetPart(target)
						if p then
							local dir = (p.Position - workspace.CurrentCamera.CFrame.Position).Unit
							for _, name in ipairs(Fruits) do
								local tool = LocalPlayer.Backpack:FindFirstChild(name) or LocalPlayer.Character:FindFirstChild(name)
								if tool then
									local remote = tool:FindFirstChild("LeftClickRemote") or tool:FindFirstChildOfClass("RemoteEvent")
									if remote then
										remote:FireServer(dir,1,true)
									end
								end
							end
						end
					end)
				end
			end
		end
	end
end)

-- BOTÃO FAST ATTACK
FastBtn.MouseButton1Click:Connect(function()
	FastAttackActive = not FastAttackActive
	FRUIT_ACTIVE = FastAttackActive -- liga/desliga junto o ataque de frutas
	FastBtn.Text = FastAttackActive and "Fast ON" or "Fast OFF"
	FastBtn.BackgroundColor3 = FastAttackActive and Color3.fromRGB(20,120,20) or Color3.fromRGB(120,10,10)
end)


--========================================================
-- WALK ON WATER (SEMPRE ATIVO)
--========================================================

task.spawn(function()
	while task.wait(0.5) do
		pcall(function()
			if workspace:FindFirstChild("Map") and workspace.Map:FindFirstChild("WaterBase-Plane") then
				local water = workspace.Map["WaterBase-Plane"]
				water.Size = Vector3.new(1000, 112, 1000) -- TAMANHO ALTO PRA ANDAR
				water.CanCollide = true
			end
		end)
	end
end)

-- =============================================
-- MISC HUB COMPLETO - BLOX FRUITS
-- =============================================

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer

-- =============================================
-- VARIÁVEIS
-- =============================================
local SpeedValor, JumpValor, FlyVelocidade = 30, 50, 50
local SpeedAtivo, JumpAtivo, FlyAtivo = false, false, false
local NoclipAtivo, AutoV4Ativo, AntiStunAtivo = false, false, false
local MenuMinimizado = true

local flyConnection, autoV4Connection, noclipConnection = nil, nil, nil
local savedCanCollide = {}

-- =============================================
-- FUNÇÕES BÁSICAS
-- =============================================
local function getCharacter()
    return player.Character or player.CharacterAdded:Wait()
end

local function getHumanoid()
    return getCharacter():FindFirstChildWhichIsA("Humanoid")
end

local function getRoot()
    local char = getCharacter()
    return char:FindFirstChild("HumanoidRootPart") or char:FindFirstChild("Torso") or char:FindFirstChild("UpperTorso")
end

-- =============================================
-- REMOVER GUI ANTIGO
-- =============================================
if player.PlayerGui:FindFirstChild("MiscHub") then
    player.PlayerGui.MiscHub:Destroy()
end

-- =============================================
-- GUI PRINCIPAL
-- =============================================
local gui = Instance.new("ScreenGui")
gui.Name = "MiscHub"
gui.ResetOnSpawn = false
gui.Parent = player:WaitForChild("PlayerGui")

local menu = Instance.new("Frame", gui)
menu.Size = UDim2.new(0,100,0,30)
menu.Position = UDim2.new(0,150,0,10)
menu.BackgroundColor3 = Color3.fromRGB(30,30,30)
menu.Active = true
menu.ClipsDescendants = true
menu.Visible = false -- começa escondido

local title = Instance.new("TextButton", menu)
title.Size = UDim2.new(1,0,0,30)
title.BackgroundColor3 = Color3.fromRGB(20,20,20)
title.Text = "MISC [+]"
title.TextColor3 = Color3.new(1,1,1)
title.Font = Enum.Font.SourceSansBold
title.TextSize = 16
title.AutoButtonColor = false

-- =============================================
-- MINIMIZAR / MAXIMIZAR
-- =============================================
title.MouseButton1Click:Connect(function()
    MenuMinimizado = not MenuMinimizado
    if MenuMinimizado then
        menu.Size = UDim2.new(0,100,0,30)
        title.Text = "MISC [+]"
    else
        menu.Size = UDim2.new(0,280,0,320)
        title.Text = "MISC - BLOX FRUITS [-]"
    end
end)

-- ARRASTAR MENU
local dragging, dragStart, startPos = false, nil, nil
title.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = menu.Position
    end
end)
title.InputEnded:Connect(function(input)
    dragging = false
end)
UserInputService.InputChanged:Connect(function(input)
    if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - dragStart
        menu.Position = UDim2.new(
            startPos.X.Scale, startPos.X.Offset + delta.X,
            startPos.Y.Scale, startPos.Y.Offset + delta.Y
        )
    end
end)

-- =============================================
-- BOTÃO FLUTUANTE PARA MOSTRAR/ESCONDER MENU
-- =============================================
MiscBtn.MouseButton1Click:Connect(function()
    if menu.Visible then
        menu.Visible = false
    else
        menu.Visible = true
        MenuMinimizado = true
        menu.Size = UDim2.new(0,100,0,30)
        title.Text = "MISC [+]"
    end
end)

-- =============================================
-- BOTÃO TOGGLE
-- =============================================
local function criarBotaoToggle(textoBase, pos, estadoInicial, callback)
    local estado = estadoInicial
    local botao = Instance.new("TextButton", menu)
    botao.Size = UDim2.new(0,120,0,30)
    botao.Position = pos
    botao.BackgroundColor3 = Color3.fromRGB(60,60,60)
    botao.TextColor3 = Color3.new(1,1,1)
    botao.Font = Enum.Font.SourceSansBold
    botao.TextSize = 14
    local function atualizarTexto()
        botao.Text = textoBase.." ["..(estado and "ON" or "OFF").."]"
    end
    atualizarTexto()
    botao.MouseButton1Click:Connect(function()
        estado = not estado
        atualizarTexto()
        callback(estado)
    end)
    return botao
end
-- =============================================
-- SLIDER
-- =============================================
local function criarSlider(nome, posY, min, max, valorInicial, callback)
    local label = Instance.new("TextLabel", menu)
    label.Size = UDim2.new(1,-20,0,20)
    label.Position = UDim2.new(0,10,0,posY)
    label.Text = nome.." ("..valorInicial..")"
    label.TextColor3 = Color3.new(1,1,1)
    label.BackgroundTransparency = 1
    label.Font = Enum.Font.SourceSansBold
    label.TextSize = 13

  local barra = Instance.new("Frame", menu)
    barra.Size = UDim2.new(1,-20,0,14)
    barra.Position = UDim2.new(0,10,0,posY+22)
    barra.BackgroundColor3 = Color3.fromRGB(70,70,70)

   local fill = Instance.new("Frame", barra)
    fill.Size = UDim2.new((valorInicial-min)/(max-min),0,1,0)
    fill.BackgroundColor3 = Color3.fromRGB(0,170,255)

   local segurando = false
    barra.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            segurando = true
        end
    end)
    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            segurando = false
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if segurando and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
            local pos = math.clamp((input.Position.X - barra.AbsolutePosition.X) / barra.AbsoluteSize.X, 0, 1)
            fill.Size = UDim2.new(pos,0,1,0)
            local value = math.floor(min + (max-min) * pos)
            label.Text = nome.." ("..value..")"
            callback(value)
        end
    end)
    return fill
end

-- SLIDERS
local fillVelocidade = criarSlider("Velocidade", 40, 30, 400, SpeedValor, function(v) SpeedValor = v end)
local fillPulo = criarSlider("Pulo", 80, 50, 400, JumpValor, function(v) JumpValor = v end)
local fillFly = criarSlider("Fly Speed", 120, 50, 1000, FlyVelocidade, function(v) FlyVelocidade = v end)

-- =============================================
-- FUNÇÕES FLY
-- =============================================
local function startFly()
    local chr, hum, root = getCharacter(), getHumanoid(), getRoot()
    if not root then return end
    hum.PlatformStand = true
    local BV = root:FindFirstChild("FlyBV") or Instance.new("BodyVelocity", root)
    BV.Name = "FlyBV"; BV.MaxForce = Vector3.new(1e9,1e9,1e9); BV.Velocity = Vector3.new(0,0,0)
    local BG = root:FindFirstChild("FlyBG") or Instance.new("BodyGyro", root)
    BG.Name = "FlyBG"; BG.MaxTorque = Vector3.new(1e9,1e9,1e9); BG.P = 9e4; BG.CFrame = root.CFrame
    flyConnection = RunService.RenderStepped:Connect(function()
        local moveDir = hum.MoveDirection
        local cam = workspace.CurrentCamera
        local localDir = cam.CFrame:VectorToObjectSpace(moveDir)
        local velocity = cam.CFrame.RightVector * localDir.X * FlyVelocidade + cam.CFrame.LookVector * (-localDir.Z) * FlyVelocidade
        BV.Velocity = velocity
        if velocity.Magnitude > 0 then
            BG.CFrame = CFrame.new(root.Position, root.Position + velocity)
        else
            BG.CFrame = CFrame.new(root.Position, root.Position + cam.CFrame.LookVector)
        end
    end)
end

local function stopFly()
    FlyAtivo = false
    if flyConnection then flyConnection:Disconnect() end
    local hum, root = getHumanoid(), getRoot()
    if hum then hum.PlatformStand = false end
    if root then
        for _, n in ipairs({"FlyBV","FlyBG"}) do
            local obj = root:FindFirstChild(n)
            if obj then obj:Destroy() end
        end
    end
end

-- =============================================
-- FUNÇÕES NOCOLIP
-- =============================================
local function AtivarNoclip()
    if noclipConnection then return end
    noclipConnection = RunService.Stepped:Connect(function()
        local char = getCharacter()
        for _,v in ipairs(char:GetDescendants()) do
            if v:IsA("BasePart") then
                if savedCanCollide[v] == nil then savedCanCollide[v] = v.CanCollide end
                v.CanCollide = false
            end
        end
    end)
end

local function DesativarNoclip()
    if noclipConnection then noclipConnection:Disconnect(); noclipConnection = nil end
    for part,orig in pairs(savedCanCollide) do
        if part and part.Parent then pcall(function() part.CanCollide = orig end) end
    end
    savedCanCollide = {}
end

-- =============================================
-- FUNÇÕES AUTO V4
-- =============================================
local function ToggleAutoV4(v)
    if v then
        local lastTime = 0
        autoV4Connection = RunService.Heartbeat:Connect(function(step)
            lastTime = lastTime + step
            if lastTime >= 1 then
                lastTime = 0
                pcall(function()
                    local Event = player.Backpack:FindFirstChild("Awakening") and player.Backpack.Awakening:FindFirstChild("RemoteFunction")
                    if Event then Event:InvokeServer(true) end
                end)
            end
        end)
    else
        if autoV4Connection then autoV4Connection:Disconnect(); autoV4Connection=nil end
    end
end

-- =============================================
-- FUNÇÕES ANTI-STUN + RESET REAL
-- =============================================
local AntiAtivo = false
local heartbeatConn, descendantConn = nil, nil
local moverClassesRagdoll = {
"BodyVelocity","BodyPosition","BodyGyro","BodyAngularVelocity",
"BodyForce","LinearVelocity","AlignPosition","AlignOrientation","VectorForce"
}

local function isMoverRagdoll(obj)
    for _, c in ipairs(moverClassesRagdoll) do
        if obj:IsA(c) then return true end
    end
    return false
end

local function limparForcasRagdoll(char)
    for _, obj in ipairs(char:GetDescendants()) do
        if isMoverRagdoll(obj) and obj.Name ~= "FlyBV" and obj.Name ~= "FlyBG" and obj.Name ~= "FlyGyro" then
            pcall(function() obj:Destroy() end)
        end
    end
end

local function forcarHumanoidRagdoll(char)
    local hum = char:FindFirstChildWhichIsA("Humanoid")
    if not hum then return end
    pcall(function()
        hum.PlatformStand = false
        hum:ChangeState(Enum.HumanoidStateType.Running)
    end)
end

local function loopRagdoll()
    if not AntiAtivo then return end
    local char = getCharacter()
    limparForcasRagdoll(char)
    forcarHumanoidRagdoll(char)
end

local function onDescAddedRagdoll(obj)
    if not AntiAtivo then return end
    if isMoverRagdoll(obj) then pcall(function() obj:Destroy() end) end
end

local function ativarRagdoll()
    if AntiAtivo then return end
    AntiAtivo = true
    local char = getCharacter()
    limparForcasRagdoll(char)
    forcarHumanoidRagdoll(char)
    descendantConn = char.DescendantAdded:Connect(onDescAddedRagdoll)
    heartbeatConn = RunService.Heartbeat:Connect(loopRagdoll)
end

local function desativarRagdoll()
    if not AntiAtivo then return end
    AntiAtivo = false
    if heartbeatConn then heartbeatConn:Disconnect(); heartbeatConn=nil end
    if descendantConn then descendantConn:Disconnect(); descendantConn=nil end
    pcall(function() player.Character:BreakJoints() end)
end

-- =============================================
-- BOTÕES
-- =============================================
local btnSpeed = criarBotaoToggle("Speed", UDim2.new(0,10,0,160), false, function(v) SpeedAtivo = v end)
local btnJump = criarBotaoToggle("Jump", UDim2.new(0,150,0,160), false, function(v)
    JumpAtivo = v
    if not v then getHumanoid().JumpPower = 50 end
end)
local btnFly = criarBotaoToggle("Fly", UDim2.new(0,10,0,200), false, function(v)
    FlyAtivo = v
    if v then startFly() else stopFly() end
end)
local btnNoclip = criarBotaoToggle("Noclip", UDim2.new(0,150,0,200), false, function(v)
    NoclipAtivo = v
    if v then AtivarNoclip() else DesativarNoclip() end
end)
local btnAutoV4 = criarBotaoToggle("Auto V4", UDim2.new(0,10,0,240), false, ToggleAutoV4)
local btnAntiStun = criarBotaoToggle("Anti-Stun", UDim2.new(0,150,0,240), false, function(v)
    if v then ativarRagdoll() else desativarRagdoll() end
end)

-- =============================================
-- LOOP PRINCIPAL
-- =============================================
RunService.RenderStepped:Connect(function()
    local hum = getHumanoid()
    if not hum then return end
    if SpeedAtivo then hum.WalkSpeed = SpeedValor end
    if JumpAtivo then hum.JumpPower = JumpValor end
end)
