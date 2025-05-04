-- LocalScript (StarterPlayerScripts)
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Workspace = game:GetService("Workspace")
local TweenService = game:GetService("TweenService")
local VirtualInputManager = game:GetService("VirtualInputManager")

local player = Players.LocalPlayer
local char = player.Character or player.CharacterAdded:Wait()
local humanoid = char:WaitForChild("Humanoid")
local hrp = char:WaitForChild("HumanoidRootPart")

local gui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
gui.Name = "AutoQuestUI"

local button = Instance.new("TextButton", gui)
button.Size = UDim2.new(0, 180, 0, 40)
button.Position = UDim2.new(0, 20, 0.5, 0)
button.Text = "Ativar Farm"
button.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
button.TextColor3 = Color3.new(1, 1, 1)
button.Font = Enum.Font.SourceSansBold
button.TextSize = 16
Instance.new("UICorner", button)

local ativo = false

-- Mover com Tween
local function moverPara(pos)
	local dist = (hrp.Position - pos).Magnitude
	local tempo = math.clamp(dist / 50, 0.5, 5)
	local tween = TweenService:Create(hrp, TweenInfo.new(tempo), {CFrame = CFrame.new(pos)})
	tween:Play()
	tween.Completed:Wait()
end

-- Aceitar missão
local function aceitarQuest()
	for _, npc in pairs(Workspace:GetDescendants()) do
		if npc:IsA("Model") and npc:FindFirstChild("Head") and npc:FindFirstChild("Humanoid") then
			if string.find(npc.Name:lower(), "quest") then
				moverPara(npc.Head.Position + Vector3.new(0, 5, 0))
				wait(1)
				local prompt = npc:FindFirstChildWhichIsA("ProximityPrompt", true)
				if prompt then
					fireproximityprompt(prompt)
					wait(1)
				end
				break
			end
		end
	end
end

-- AutoClick usando soco
local function autoClick()
	while ativo do
		local tool = player.Character and player.Character:FindFirstChildOfClass("Tool")
		if tool then
			tool:Activate()
		else
			-- Alternativamente, emula clique primário
			VirtualInputManager:SendMouseButtonEvent(500, 500, 0, true, game, 0)
			VirtualInputManager:SendMouseButtonEvent(500, 500, 0, false, game, 0)
		end
		wait(0.1)
	end
end

-- Atacar de cima
local function atacarInimigos()
	while ativo do
		for _, npc in pairs(Workspace:GetDescendants()) do
			if npc:IsA("Model") and npc:FindFirstChild("Humanoid") and npc:FindFirstChild("HumanoidRootPart") and npc.Humanoid.Health > 0 then
				if string.find(npc.Name:lower(), "bandit") or string.find(npc.Name:lower(), "enemy") then
					local targetPos = npc.HumanoidRootPart.Position + Vector3.new(0, 10, 0)
					moverPara(targetPos)
					while npc and npc.Parent and npc.Humanoid.Health > 0 and ativo do
						wait(0.2)
					end
				end
			end
		end
		wait(1)
	end
end

-- Botão toggle
button.MouseButton1Click:Connect(function()
	ativo = not ativo
	button.Text = ativo and "Parar Farm" or "Ativar Farm"
	if ativo then
		aceitarQuest()
		task.spawn(autoClick)
		task.spawn(atacarInimigos)
	end
end)
