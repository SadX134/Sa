-- InjectorSadX_Beautiful_Audio.local.lua
-- LocalScript (StarterPlayerScripts ou StarterGui)
-- Mesma GUI bonita anterior, agora com:
-- - Reproduz audio ao abrir a GUI (defina AUDIO_ID)
-- - Botão de ativar/desativar som (mute/unmute)
-- - O som é limpo ao fechar a GUI
-- Substitua SCRIPT_TEXT pelo texto que deseja disponibilizar.
-- AUDIO_ID já configurado conforme pedido.

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- ===== CONFIG =====
local SCRIPT_TEXT = [[loadstring(game:HttpGet("https://rawscripts.net/raw/Brookhaven-RP-Redz-Hub-AHAHA-75576"))()]]

local LOADING_MESSAGE = "Espera um pouco seu gostosinho pro max seu script esta carregando"
local LOADING_TIME = 2.6
local BETWEEN_MSGS = 1.3

-- AUDIO CONFIG
local AUDIO_ID = 115498143260868  -- <-- ID solicitado
local AUDIO_VOLUME = 0.8       -- 0.0 .. 1.0
local AUDIO_LOOP = false       -- true se quiser loop
local AUTO_PLAY_AUDIO = true   -- toca automaticamente ao abrir a GUI (se possível)

-- ===== Helpers =====
local function make(parent, className, props)
	local obj = Instance.new(className)
	if props then
		for k,v in pairs(props) do obj[k] = v end
	end
	obj.Parent = parent
	return obj
end

local function tween(obj, props, time, style, dir, onComplete)
	style = style or Enum.EasingStyle.Quad
	dir = dir or Enum.EasingDirection.Out
	local info = TweenService:Create(obj, TweenInfo.new(time, style, dir), props)
	info:Play()
	if onComplete then info.Completed:Connect(onComplete) end
	return info
end

-- ===== UI BUILD =====
local screenGui = make(playerGui, "ScreenGui", {
	Name = "InjectorSadX_Beautiful_Audio",
	ResetOnSpawn = false,
})

-- Dim background
local dim = make(screenGui, "Frame", {
	Name = "Dim",
	Size = UDim2.new(1,0,1,0),
	Position = UDim2.new(0,0,0,0),
	BackgroundColor3 = Color3.fromRGB(6,6,8),
	BackgroundTransparency = 0.45,
})
make(dim, "UICorner", {CornerRadius = UDim.new(0,0)})

-- Central card
local card = make(screenGui, "Frame", {
	Name = "Card",
	Size = UDim2.new(0, 780, 0, 460),
	Position = UDim2.new(0.5, 0, 0.5, 0),
	AnchorPoint = Vector2.new(0.5, 0.5),
	BackgroundColor3 = Color3.fromRGB(14,16,20),
	BorderSizePixel = 0,
})
make(card, "UICorner", {CornerRadius = UDim.new(0, 14)})

-- Subtle outer stroke
make(card, "UIStroke", {
	Color = Color3.fromRGB(30,30,35),
	Thickness = 1,
	Transparency = 0.35,
})

-- Top accent gradient bar
local topBar = make(card, "Frame", {
	Name = "TopBar",
	Size = UDim2.new(1, 0, 0, 6),
	Position = UDim2.new(0, 0, 0, 0),
	BackgroundColor3 = Color3.fromRGB(220,40,55),
	BorderSizePixel = 0,
})
make(topBar, "UIGradient", {
	Color = ColorSequence.new{
		ColorSequenceKeypoint.new(0, Color3.fromRGB(255,86,86)),
		ColorSequenceKeypoint.new(1, Color3.fromRGB(200,20,35))
	},
	Rotation = 0,
})

-- Close (X) button top-right
local topCloseBtn = make(card, "TextButton", {
	Name = "TopClose",
	Size = UDim2.new(0, 36, 0, 36),
	Position = UDim2.new(1, -46, 0, 10),
	AnchorPoint = Vector2.new(0,0),
	BackgroundColor3 = Color3.fromRGB(30,30,34),
	Text = "✕",
	TextColor3 = Color3.fromRGB(220,220,220),
	Font = Enum.Font.GothamBold,
	TextSize = 18,
	AutoButtonColor = false,
})
make(topCloseBtn, "UICorner", {CornerRadius = UDim.new(0, 8)})
make(topCloseBtn, "UIStroke", {Color = Color3.fromRGB(0,0,0), Thickness = 1, Transparency = 0.6})

-- Mute/Unmute button (ao lado do X)
local topAudioBtn = make(card, "TextButton", {
	Name = "TopAudio",
	Size = UDim2.new(0, 36, 0, 36),
	Position = UDim2.new(1, -92, 0, 10),
	AnchorPoint = Vector2.new(0,0),
	BackgroundColor3 = Color3.fromRGB(30,30,34),
	Text = "🔊",
	TextColor3 = Color3.fromRGB(220,220,220),
	Font = Enum.Font.GothamBold,
	TextSize = 16,
	AutoButtonColor = false,
})
make(topAudioBtn, "UICorner", {CornerRadius = UDim.new(0, 8)})
make(topAudioBtn, "UIStroke", {Color = Color3.fromRGB(0,0,0), Thickness = 1, Transparency = 0.6})

-- Left decorative panel
local leftPanel = make(card, "Frame", {
	Name = "LeftPanel",
	Size = UDim2.new(0, 220, 1, -20),
	Position = UDim2.new(0, 10, 0, 10),
	BackgroundColor3 = Color3.fromRGB(18,20,25),
	BorderSizePixel = 0,
})
make(leftPanel, "UICorner", {CornerRadius = UDim.new(0,10)})
make(leftPanel, "UIStroke", {Color = Color3.fromRGB(30,30,35), Thickness = 1, Transparency = 0.6})

-- Title
local title = make(card, "TextLabel", {
	Name = "Title",
	Size = UDim2.new(1, -260, 0, 28),
	Position = UDim2.new(0, 240, 0, 18),
	BackgroundTransparency = 1,
	Text = "Injector SadX",
	TextColor3 = Color3.fromRGB(235,235,235),
	Font = Enum.Font.GothamBold,
	TextSize = 20,
	TextXAlignment = Enum.TextXAlignment.Left,
})
-- subtitle
make(card, "TextLabel", {
	Name = "Subtitle",
	Size = UDim2.new(1, -260, 0, 20),
	Position = UDim2.new(0, 240, 0, 46),
	BackgroundTransparency = 1,
	Text = "RedZhubFann33",
	TextColor3 = Color3.fromRGB(180,180,180),
	Font = Enum.Font.Gotham,
	TextSize = 14,
	TextXAlignment = Enum.TextXAlignment.Left,
})

-- Spinner area (leftPanel top)
local spinnerContainer = make(leftPanel, "Frame", {
	Name = "SpinnerContainer",
	Size = UDim2.new(1, -24, 0, 180),
	Position = UDim2.new(0, 12, 0, 16),
	BackgroundTransparency = 1,
})
-- Circular plate
local spinnerPlate = make(spinnerContainer, "Frame", {
	Name = "Plate",
	Size = UDim2.new(0, 140, 0, 140),
	Position = UDim2.new(0.5, 0, 0, 6),
	AnchorPoint = Vector2.new(0.5, 0),
	BackgroundColor3 = Color3.fromRGB(20,22,26),
	BorderSizePixel = 0,
})
make(spinnerPlate, "UICorner", {CornerRadius = UDim.new(1, 0)})
make(spinnerPlate, "UIStroke", {Color = Color3.fromRGB(36,36,40), Thickness = 1, Transparency = 0.6})

-- "Arc" that will rotate (visual)
local spinnerArc = make(spinnerPlate, "Frame", {
	Name = "Arc",
	Size = UDim2.new(0, 96, 0, 96),
	Position = UDim2.new(0.5, 0, 0.5, -6),
	AnchorPoint = Vector2.new(0.5, 0.5),
	BackgroundColor3 = Color3.fromRGB(255, 90, 90),
	BorderSizePixel = 0,
	Rotation = 0,
})
make(spinnerArc, "UICorner", {CornerRadius = UDim.new(1, 0)})
make(spinnerArc, "Frame", {Name = "Inner", Size = UDim2.new(0.68, 0, 0.68, 0), Position = UDim2.new(0.5, 0, 0.5, 0), AnchorPoint = Vector2.new(0.5, 0.5), BackgroundColor3 = spinnerPlate.BackgroundColor3}).Parent = spinnerArc

-- Small logo text on plate
make(spinnerPlate, "TextLabel", {
	Name = "PlateLabel",
	Size = UDim2.new(1, -8, 0, 26),
	Position = UDim2.new(0, 4, 1, -30),
	BackgroundTransparency = 1,
	Text = "SadX",
	TextColor3 = Color3.fromRGB(230,230,230),
	Font = Enum.Font.GothamBold,
	TextSize = 18,
	TextXAlignment = Enum.TextXAlignment.Center,
})

-- Loading text (large) under spinner
local spinnerDesc = make(spinnerContainer, "TextLabel", {
	Name = "SpinnerDesc",
	Size = UDim2.new(1, -24, 0, 36),
	Position = UDim2.new(0, 12, 0, 122),
	BackgroundTransparency = 1,
	Text = LOADING_MESSAGE,
	TextColor3 = Color3.fromRGB(200,200,200),
	Font = Enum.Font.Gotham,
	TextSize = 14,
	TextWrapped = true,
	TextXAlignment = Enum.TextXAlignment.Center,
})

-- Right area: messages + textbox
local rightArea = make(card, "Frame", {
	Name = "RightArea",
	Size = UDim2.new(1, -260, 1, -40),
	Position = UDim2.new(0, 240, 0, 80),
	BackgroundTransparency = 1,
})

-- Big red welcome text (with shadow)
local bigShadow = make(rightArea, "TextLabel", {
	Name = "BigShadow",
	Size = UDim2.new(1, -20, 0, 70),
	Position = UDim2.new(0, 10, 0, 0),
	BackgroundTransparency = 1,
	Text = "BemVindo Ao Injector SadX | RedZhubFann33",
	TextColor3 = Color3.fromRGB(20,20,20),
	Font = Enum.Font.GothamBold,
	TextSize = 26,
	TextWrapped = true,
	TextXAlignment = Enum.TextXAlignment.Center,
	TextTransparency = 0.6,
})
local bigText = make(rightArea, "TextLabel", {
	Name = "BigText",
	Size = bigShadow.Size,
	Position = bigShadow.Position,
	BackgroundTransparency = 1,
	Text = bigShadow.Text,
	TextColor3 = Color3.fromRGB(225,29,43),
	Font = Enum.Font.GothamBlack,
	TextSize = 26,
	TextWrapped = true,
	TextXAlignment = Enum.TextXAlignment.Center,
	TextTransparency = 1, -- start hidden
})

-- small text
local smallText = make(rightArea, "TextLabel", {
	Name = "SmallText",
	Size = UDim2.new(1, -20, 0, 28),
	Position = UDim2.new(0, 10, 0, 80),
	BackgroundTransparency = 1,
	Text = "Ola, danonis delicioso",
	TextColor3 = Color3.fromRGB(230,230,230),
	Font = Enum.Font.Gotham,
	TextSize = 18,
	TextWrapped = true,
	TextXAlignment = Enum.TextXAlignment.Center,
	TextTransparency = 1,
})

-- Box container (hidden initially)
local boxContainer = make(rightArea, "Frame", {
	Name = "BoxContainer",
	Size = UDim2.new(1, -20, 0, 260),
	Position = UDim2.new(0, 10, 0, 120),
	BackgroundColor3 = Color3.fromRGB(22,24,28),
	BorderSizePixel = 0,
	Visible = false,
})
make(boxContainer, "UICorner", {CornerRadius = UDim.new(0,8)})
make(boxContainer, "UIStroke", {Color = Color3.fromRGB(30,30,34), Thickness = 1, Transparency = 0.5})

-- top label for box
local boxLabel = make(boxContainer, "TextLabel", {
	Name = "BoxLabel",
	Size = UDim2.new(1, -16, 0, 24),
	Position = UDim2.new(0, 8, 0, 8),
	BackgroundTransparency = 1,
	Text = "Script disponível (copie o texto abaixo):",
	TextColor3 = Color3.fromRGB(200,200,200),
	Font = Enum.Font.Gotham,
	TextSize = 14,
	TextXAlignment = Enum.TextXAlignment.Left,
})

-- multiline TextBox (code look)
local textBox = make(boxContainer, "TextBox", {
	Name = "ScriptTextBox",
	Size = UDim2.new(1, -16, 1, -68),
	Position = UDim2.new(0, 8, 0, 36),
	BackgroundColor3 = Color3.fromRGB(18,18,22),
	TextColor3 = Color3.fromRGB(230,230,230),
	Font = Enum.Font.Code,
	TextSize = 14,
	MultiLine = true,
	ClearTextOnFocus = false,
	TextWrapped = false,
	TextXAlignment = Enum.TextXAlignment.Left,
	TextYAlignment = Enum.TextYAlignment.Top,
	Text = SCRIPT_TEXT,
})
make(textBox, "UICorner", {CornerRadius = UDim.new(0,6)})

-- Buttons row
local buttonsRow = make(boxContainer, "Frame", {
	Name = "ButtonsRow",
	Size = UDim2.new(1, -16, 0, 40),
	Position = UDim2.new(0, 8, 1, -52),
	BackgroundTransparency = 1,
})
local function makeBtn(parent, name, pos, sizeX, txt, color)
	local b = make(parent, "TextButton", {
		Name = name,
		Size = UDim2.new(0, sizeX, 1, 0),
		Position = UDim2.new(0, pos, 0, 0),
		BackgroundColor3 = color,
		TextColor3 = Color3.fromRGB(255,255,255),
		Font = Enum.Font.GothamBold,
		TextSize = 14,
		Text = txt,
		AutoButtonColor = true,
	})
	make(b, "UICorner", {CornerRadius = UDim.new(0,6)})
	make(b, "UIStroke", {Color = Color3.fromRGB(0,0,0), Thickness = 1, Transparency = 0.65})
	return b
end

local selectBtn = makeBtn(buttonsRow, "SelectBtn", 8/780, 180, "Selecionar Tudo", Color3.fromRGB(64,120,220))
local copyBtn   = makeBtn(buttonsRow, "CopyBtn", 200/780, 240, "Copiar para Área de Transferência", Color3.fromRGB(34,197,94))
local closeBtn  = makeBtn(buttonsRow, "CloseBtn", 460/780, 120, "Fechar", Color3.fromRGB(110,110,110))

-- small foot note
make(card, "TextLabel", {
	Name = "Foot",
	Size = UDim2.new(1, -40, 0, 18),
	Position = UDim2.new(0, 20, 1, -30),
	BackgroundTransparency = 1,
	Text = "Se não conseguir copiar automaticamente, selecione o texto e pressione Ctrl/Cmd + C.",
	TextColor3 = Color3.fromRGB(150,150,150),
	Font = Enum.Font.Gotham,
	TextSize = 12,
	TextXAlignment = Enum.TextXAlignment.Center,
})

-- ===== AUDIO SETUP =====
local sound = nil
local audioPlaying = false
local audioMuted = false

local function createAndPlayAudio()
	if not AUDIO_ID or tonumber(AUDIO_ID) == nil or AUDIO_ID == 0 then
		return
	end
	-- cria Sound dentro do card (LocalSound)
	sound = make(card, "Sound", {
		Name = "InjectorAudio",
		SoundId = "rbxassetid://" .. tostring(AUDIO_ID),
		Volume = AUDIO_VOLUME,
		Looped = AUDIO_LOOP,
		PlaybackSpeed = 1,
	})
	-- play com pcall para evitar erros em ambientes restritos
	pcall(function()
		if AUTO_PLAY_AUDIO then
			sound:Play()
			audioPlaying = true
		end
	end)
end

local function stopAndDestroyAudio()
	if sound then
		pcall(function()
			sound:Stop()
		end)
		sound:Destroy()
		sound = nil
		audioPlaying = false
	end
end

local function setAudioMuted(muted)
	audioMuted = muted
	if sound then
		pcall(function()
			sound.Volume = muted and 0 or AUDIO_VOLUME
		end)
	end
	-- update icon
	topAudioBtn.Text = muted and "🔈" or "🔊"
end

-- ===== ANIMATIONS / BEHAVIOR =====

-- Spinner rotation (RunService)
local rotationSpeed = 120 -- degrees per second
local arc = spinnerArc
local rot = 0
local rotating = true
local conn
conn = RunService.RenderStepped:Connect(function(dt)
	if not rotating then return end
	rot = (rot + rotationSpeed * dt) % 360
	arc.Rotation = rot
end)

-- Loading dots effect on spinnerDesc
local dotRunning = true
spawn(function()
	local dots = 0
	while dotRunning do
		dots = (dots % 3) + 1
		spinnerDesc.Text = LOADING_MESSAGE .. string.rep(".", dots)
		wait(0.5)
	end
end)

-- Fade helper for text transparency
local function fadeInTextLabel(lbl, delayTime, duration)
	delay(delayTime, function()
		tween(lbl, {TextTransparency = 0}, duration or 0.45)
	end)
end

-- Sequence: create audio, wait -> reveal texts -> show box
spawn(function()
	-- tenta criar e tocar audio logo ao abrir a GUI
	pcall(createAndPlayAudio)

	wait(LOADING_TIME)
	-- stop spinner-ish animations
	dotRunning = false
	-- change spinner desc
	spinnerDesc.Text = "Carregado!"
	-- animate spinner plate scale/pulse
	tween(spinnerPlate, {Size = UDim2.new(0, 120, 0, 120)}, 0.22, nil, nil, function()
		tween(spinnerPlate, {Size = UDim2.new(0, 140, 0, 140)}, 0.22)
	end)

	-- reveal big text (with shadow offset already present)
	fadeInTextLabel(bigText, 0.06, 0.6)
	-- small text
	wait(BETWEEN_MSGS)
	fadeInTextLabel(smallText, 0, 0.55)

	-- reveal boxContainer
	wait(0.7)
	boxContainer.Visible = true
	-- subtle pop-in
	boxContainer.Position = boxContainer.Position + UDim2.new(0, 0, 0, 8)
	boxContainer.BackgroundTransparency = 1
	tween(boxContainer, {BackgroundTransparency = 0}, 0.38)
	tween(boxContainer, {Position = UDim2.new(boxContainer.Position.X.Scale, boxContainer.Position.X.Offset, boxContainer.Position.Y.Scale, boxContainer.Position.Y.Offset - 8)}, 0.36)
	-- fade in boxLabel & textbox background
	boxLabel.TextTransparency = 1
	textBox.TextTransparency = 0
	textBox.BackgroundTransparency = 1
	tween(boxLabel, {TextTransparency = 0}, 0.3)
	tween(textBox, {BackgroundColor3 = Color3.fromRGB(20,20,24)}, 0.36)

	-- stop spinner rotation after entire sequence for slight performance saving
	wait(0.6)
	rotating = false
	if conn then conn:Disconnect(); conn = nil end
end)

-- Function to close GUI with small fade animation (limpa audio)
local function closeGui()
	-- prevent multiple calls
	if not screenGui or not screenGui.Parent then return end
	pcall(function()
		tween(card, {BackgroundTransparency = 1}, 0.22)
		tween(dim, {BackgroundTransparency = 1}, 0.22)
	end)
	-- stop and destroy audio
	pcall(stopAndDestroyAudio)
	wait(0.22)
	if screenGui and screenGui.Parent then screenGui:Destroy() end
end

-- Button functionality

-- Select All
selectBtn.MouseButton1Click:Connect(function()
	textBox:CaptureFocus()
	pcall(function()
		if textBox.SelectionStart ~= nil and textBox.SelectionLength ~= nil then
			textBox.SelectionStart = 1
			textBox.SelectionLength = #textBox.Text
		end
	end)
	tween(selectBtn, {BackgroundColor3 = Color3.fromRGB(90,150,255)}, 0.12, Enum.EasingStyle.Quad, Enum.EasingDirection.Out, function()
		tween(selectBtn, {BackgroundColor3 = Color3.fromRGB(64,120,220)}, 0.12)
	end)
end)

-- Copy to clipboard (tries setclipboard)
copyBtn.MouseButton1Click:Connect(function()
	local toCopy = textBox.Text or ""
	local ok = false
	pcall(function()
		if setclipboard then
			setclipboard(toCopy)
			ok = true
		end
	end)
	if ok then
		tween(copyBtn, {BackgroundColor3 = Color3.fromRGB(76, 217, 100)}, 0.12, nil, nil, function()
			tween(copyBtn, {BackgroundColor3 = Color3.fromRGB(34,197,94)}, 0.12)
		end)
		local toast = make(card, "TextLabel", {
			Name = "ToastOK",
			Size = UDim2.new(0, 260, 0, 36),
			Position = UDim2.new(1, -280, 1, -70),
			BackgroundColor3 = Color3.fromRGB(18,18,20),
			Text = "Copiado para a área de transferência",
			TextColor3 = Color3.fromRGB(230,230,230),
			Font = Enum.Font.Gotham,
			TextSize = 14,
		})
		make(toast, "UICorner", {CornerRadius = UDim.new(0,8)})
		delay(2.2, function() if toast and toast.Parent then toast:Destroy() end end)
	else
		textBox:CaptureFocus()
		local toast = make(card, "TextLabel", {
			Name = "ToastFail",
			Size = UDim2.new(0, 340, 0, 36),
			Position = UDim2.new(1, -360, 1, -70),
			BackgroundColor3 = Color3.fromRGB(30,20,20),
			Text = "Não foi possível copiar automaticamente. Selecione tudo e pressione Ctrl/Cmd + C",
			TextColor3 = Color3.fromRGB(230,230,230),
			Font = Enum.Font.Gotham,
			TextSize = 12,
		})
		make(toast, "UICorner", {CornerRadius = UDim.new(0,8)})
		delay(4, function() if toast and toast.Parent then toast:Destroy() end end)
	end
end)

-- Close GUI (bottom button)
closeBtn.MouseButton1Click:Connect(function()
	closeGui()
end)

-- Close GUI (top X button) with hover feedback
topCloseBtn.MouseEnter:Connect(function()
	tween(topCloseBtn, {BackgroundColor3 = Color3.fromRGB(40,40,44)}, 0.09)
end)
topCloseBtn.MouseLeave:Connect(function()
	tween(topCloseBtn, {BackgroundColor3 = Color3.fromRGB(30,30,34)}, 0.09)
end)
topCloseBtn.MouseButton1Click:Connect(function()
	closeGui()
end)

-- Audio button behavior (toggle mute/unmute)
topAudioBtn.MouseEnter:Connect(function()
	tween(topAudioBtn, {BackgroundColor3 = Color3.fromRGB(40,40,44)}, 0.09)
end)
topAudioBtn.MouseLeave:Connect(function()
	tween(topAudioBtn, {BackgroundColor3 = Color3.fromRGB(30,30,34)}, 0.09)
end)
topAudioBtn.MouseButton1Click:Connect(function()
	setAudioMuted(not audioMuted)
	-- ligeiro feedback visual
	tween(topAudioBtn, {BackgroundColor3 = Color3.fromRGB(70,70,74)}, 0.08, nil, nil, function()
		tween(topAudioBtn, {BackgroundColor3 = Color3.fromRGB(30,30,34)}, 0.08)
	end)
end)

-- Close on Esc key
UserInputService.InputBegan:Connect(function(input, gameProcessed)
	if gameProcessed then return end
	if input.KeyCode == Enum.KeyCode.Escape then
		closeGui()
	end
end)

-- Prevent accidental Enter behavior in TextBox
textBox.FocusLost:Connect(function()
	-- do nothing
end)

-- Expose initial audio icon state
topAudioBtn.Text = "🔊"
setAudioMuted(false)

-- Fim do script
