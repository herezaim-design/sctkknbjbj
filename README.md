-- [[ HUD Editor para Mobile - versão autônoma e funcional ]]
-- Coloque este script como LocalScript em StarterPlayerScripts

local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer

-- Cria GUI base
local gui = Instance.new("ScreenGui")
gui.Name = "MobileHUD"
gui.ResetOnSpawn = false
gui.IgnoreGuiInset = true
gui.Parent = player:WaitForChild("PlayerGui")

-- Função para criar botões padrão
local function makeButton(name, text, pos)
	local b = Instance.new("ImageButton")
	b.Name = name
	b.AnchorPoint = Vector2.new(0.5, 0.5)
	b.Size = UDim2.new(0.15, 0, 0.09, 0)
	b.Position = pos
	b.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
	b.BackgroundTransparency = 0.1
	b.AutoButtonColor = true
	b.Parent = gui

	local lbl = Instance.new("TextLabel")
	lbl.BackgroundTransparency = 1
	lbl.Size = UDim2.new(1, 0, 1, 0)
	lbl.TextColor3 = Color3.new(1, 1, 1)
	lbl.TextScaled = true
	lbl.Text = text
	lbl.Parent = b

	return b
end

-- Cria os botões principais
local playBtn = makeButton("PlayButton", "PLAY", UDim2.new(0.5, 0, 0.85, 0))
local menuBtn = makeButton("MenuButton", "MENU", UDim2.new(0.15, 0, 0.9, 0))

-- Painel lateral simples
local panel = Instance.new("Frame")
panel.Name = "EditorPanel"
panel.AnchorPoint = Vector2.new(1, 0)
panel.Size = UDim2.new(0, 140, 0, 110)
panel.Position = UDim2.new(1, -10, 0, 10)
panel.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
panel.BackgroundTransparency = 0.15
panel.Parent = gui

local function makePanelButton(text, y)
	local b = Instance.new("TextButton")
	b.Size = UDim2.new(1, -10, 0, 28)
	b.Position = UDim2.new(0, 5, 0, y)
	b.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
	b.TextColor3 = Color3.new(1, 1, 1)
	b.Text = text
	b.Parent = panel
	return b
end

local editBtn = makePanelButton("Entrar Modo Edição", 5)
local saveBtn = makePanelButton("Salvar", 38)
local resetBtn = makePanelButton("Resetar", 71)

-- Estado de edição
local editing = false
local dragging = false
local resizing = false
local selected = nil
local dragOffset = Vector2.new()

-- Retângulo de seleção visual
local selectionRect = Instance.new("Frame")
selectionRect.BorderSizePixel = 2
selectionRect.BorderColor3 = Color3.fromRGB(255, 255, 0)
selectionRect.BackgroundTransparency = 1
selectionRect.ZIndex = 10
selectionRect.Visible = false
selectionRect.Parent = gui

local function updateSelection(target)
	if target then
		selectionRect.Visible = true
		selectionRect.Position = target.Position
		selectionRect.Size = target.Size
		selectionRect.AnchorPoint = target.AnchorPoint
	else
		selectionRect.Visible = false
	end
end

-- Alterna modo de edição
editBtn.Activated:Connect(function()
	editing = not editing
	editBtn.Text = editing and "Sair do Modo" or "Entrar Modo Edição"
	updateSelection(nil)
end)

-- Funções auxiliares
local function clamp(v, a, b)
	return math.max(a, math.min(b, v))
end

-- Detecta início de toque
local function startTouch(btn, input)
	if not editing then return end
	selected = btn
	updateSelection(btn)
	local abs = btn.AbsolutePosition
	local size = btn.AbsoluteSize
	local pos = input.Position
	local br = abs + size

	local dx, dy = math.abs(pos.X - br.X), math.abs(pos.Y - br.Y)
	if dx < 30 and dy < 30 then
		resizing = true
	else
		dragging = true
		dragOffset = pos - (abs + size / 2)
	end

	input.Changed:Connect(function()
		if input.UserInputState == Enum.UserInputState.Ended then
			dragging, resizing = false, false
		end
	end)
end

-- Conecta eventos de toque/mouse
local function attach(btn)
	btn.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
			startTouch(btn, input)
		end
	end)
end

attach(playBtn)
attach(menuBtn)

UIS.InputChanged:Connect(function(input)
	if not editing or not selected then return end
	if input.UserInputType ~= Enum.UserInputType.Touch and input.UserInputType ~= Enum.UserInputType.MouseMovement then
		return
	end

	local pos = input.Position
	local viewport = workspace.CurrentCamera.ViewportSize
	if dragging then
		local center = pos - dragOffset
		selected.Position = UDim2.new(clamp(center.X / viewport.X, 0.05, 0.95), 0, clamp(center.Y / viewport.Y, 0.05, 0.95), 0)
	elseif resizing then
		local abs = selected.AbsolutePosition
		local newSize = Vector2.new(pos.X - abs.X, pos.Y - abs.Y)
		selected.Size = UDim2.new(clamp(newSize.X / viewport.X, 0.05, 0.4), 0, clamp(newSize.Y / viewport.Y, 0.05, 0.3), 0)
	end
	updateSelection(selected)
end)

-- Atualiza seleção visual
RunService.RenderStepped:Connect(function()
	if selected and editing then
		updateSelection(selected)
	end
end)

-- Reset layout padrão
resetBtn.Activated:Connect(function()
	playBtn.Position = UDim2.new(0.5, 0, 0.85, 0)
	playBtn.Size = UDim2.new(0.15, 0, 0.09, 0)
	menuBtn.Position = UDim2.new(0.15, 0, 0.9, 0)
	menuBtn.Size = UDim2.new(0.15, 0, 0.09, 0)
end)

-- "Salvar" (neste exemplo, só printa no Output)
saveBtn.Activated:Connect(function()
	print("Layout salvo!")
	print("Play:", playBtn.Position, playBtn.Size)
	print("Menu:", menuBtn.Position, menuBtn.Size)
end)

-- Ações originais simuladas
playBtn.Activated:Connect(function()
	if editing then return end
	print("Ação: PLAY")
end)
menuBtn.Activated:Connect(function()
	if editing then return end
	print("Ação: MENU")
end)
