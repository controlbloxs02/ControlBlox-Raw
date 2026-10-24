--[[
    C O N T R O L   B L O X   —   v4
    LocalScript -> StarterPlayer > StarterPlayerScripts

    Abrir/fechar: botão flutuante CB (arrastável) ou RightShift

    API:
      local Window = Library:CreateWindow({ Name, Version, ToggleKey, Intro, Accent, Volume, Pitch })
      local Tab    = Window:AddTab("Nome")
      Tab:AddSection("Título")
      Tab:AddLabel("Texto")
      Tab:AddButton("Texto", function() end)
      Tab:AddToggle("Texto", false, function(on) end)
      Tab:AddSlider("Texto", min, max, padrão, function(v) end)
      Tab:AddDropdown("Texto", {"A","B"}, "A", function(opcao) end)
      Tab:AddTextBox("Texto", "placeholder", function(texto) end)
      Tab:AddKeybind("Texto", Enum.KeyCode.F, function() end)
      Tab:AddColorPicker("Texto", Color3, function(cor) end)
      Window:AddProfileTab("Perfil")  -- aba com avatar inteiro e dados da conta
      Window:Notify("Título", "Mensagem", segundos)
      Window:SetAccent(Color3)        -- cor de destaque
      Window:SetVolume(0..1)          -- volume dos sons
      Window:SetPitch(0.5..2)         -- tom (altura) dos sons
      Window:SetButtonVisible(true)   -- botão flutuante
      Window:PlaySound("click")       -- tocar um som do menu
      Window:RefreshProfile()         -- recalcula valor do avatar
]]

local Players            = game:GetService("Players")
local TweenService       = game:GetService("TweenService")
local UserInputService   = game:GetService("UserInputService")
local RunService         = game:GetService("RunService")
local MarketplaceService = game:GetService("MarketplaceService")
local Stats              = game:GetService("Stats")

local LocalPlayer = Players.LocalPlayer

--------------------------------------------------------------------
-- TEMA
--------------------------------------------------------------------
local Theme = {
	Black   = Color3.fromRGB(5, 5, 7),
	Bg      = Color3.fromRGB(11, 11, 14),
	Surface = Color3.fromRGB(18, 18, 23),
	Hover   = Color3.fromRGB(26, 26, 33),
	Line    = Color3.fromRGB(36, 36, 46),
	Accent  = Color3.fromRGB(108, 32, 255), -- muda com Window:SetAccent
	Text    = Color3.fromRGB(245, 245, 250),
	Sub     = Color3.fromRGB(120, 120, 138),
	Good    = Color3.fromRGB(80, 220, 140),
	Warn    = Color3.fromRGB(255, 200, 60),
	Bad     = Color3.fromRGB(255, 80, 100),
	Title   = Enum.Font.GothamBlack,
	Bold    = Enum.Font.GothamBold,
	Body    = Enum.Font.Gotham,
	Mono    = Enum.Font.Code,
}

--------------------------------------------------------------------
-- DESIGNS / PRESETS VISUAIS
--------------------------------------------------------------------
local Designs = {
    DarkPurple = {
        Black = Color3.fromRGB(5, 5, 8), Bg = Color3.fromRGB(11, 11, 16),
        Surface = Color3.fromRGB(20, 18, 29), Hover = Color3.fromRGB(35, 28, 48),
        Line = Color3.fromRGB(53, 43, 68), Accent = Color3.fromRGB(132, 55, 255),
        Text = Color3.fromRGB(246, 242, 255), Sub = Color3.fromRGB(155, 143, 176),
    },
    Midnight = {
        Black = Color3.fromRGB(3, 6, 12), Bg = Color3.fromRGB(8, 13, 22),
        Surface = Color3.fromRGB(15, 24, 38), Hover = Color3.fromRGB(24, 37, 55),
        Line = Color3.fromRGB(37, 53, 73), Accent = Color3.fromRGB(42, 150, 255),
        Text = Color3.fromRGB(238, 246, 255), Sub = Color3.fromRGB(135, 157, 180),
    },
    Crimson = {
        Black = Color3.fromRGB(10, 4, 6), Bg = Color3.fromRGB(18, 8, 11),
        Surface = Color3.fromRGB(30, 14, 18), Hover = Color3.fromRGB(48, 20, 28),
        Line = Color3.fromRGB(70, 34, 43), Accent = Color3.fromRGB(255, 50, 80),
        Text = Color3.fromRGB(255, 242, 245), Sub = Color3.fromRGB(180, 140, 150),
    },
    Original = {
        Black = Color3.fromRGB(5, 5, 7), Bg = Color3.fromRGB(11, 11, 14),
        Surface = Color3.fromRGB(18, 18, 23), Hover = Color3.fromRGB(26, 26, 33),
        Line = Color3.fromRGB(36, 36, 46), Accent = Color3.fromRGB(108, 32, 255),
        Text = Color3.fromRGB(245, 245, 250), Sub = Color3.fromRGB(120, 120, 138),
    },
}

--------------------------------------------------------------------
-- SONS  (troque os IDs por "rbxassetid://NUMERO" se quiser outros)
-- Estes usam sons que já vêm dentro do próprio Roblox.
--------------------------------------------------------------------
local Sounds = {
	open   = { id = "rbxasset://sounds/switch.wav",             vol = 0.9, speed = 1.00 },
	close  = { id = "rbxasset://sounds/switch.wav",             vol = 0.9, speed = 0.80 },
	click  = { id = "rbxasset://sounds/button.wav",             vol = 0.8, speed = 1.20 },
	tab    = { id = "rbxasset://sounds/button.wav",             vol = 0.7, speed = 1.40 },
	toggle = { id = "rbxasset://sounds/switch.wav",             vol = 0.9, speed = 1.00 },
	slider = { id = "rbxasset://sounds/button.wav",             vol = 0.6, speed = 1.60 },
	notify = { id = "rbxasset://sounds/electronicpingshort.wav", vol = 1.2, speed = 1.00 },
}

--------------------------------------------------------------------
-- HELPERS
--------------------------------------------------------------------
local function new(class, props, children)
	local o = Instance.new(class)
	for k, v in pairs(props or {}) do o[k] = v end
	for _, c in ipairs(children or {}) do c.Parent = o end
	return o
end

local function corner(r) return new("UICorner", { CornerRadius = UDim.new(0, r or 6) }) end

local function stroke(color, thickness, transparency)
	return new("UIStroke", {
		Color = color or Theme.Line,
		Thickness = thickness or 1,
		Transparency = transparency or 0,
		ApplyStrokeMode = Enum.ApplyStrokeMode.Border,
	})
end

local function tween(obj, t, props, style, dir)
	local tw = TweenService:Create(obj, TweenInfo.new(t, style or Enum.EasingStyle.Quint, dir or Enum.EasingDirection.Out), props)
	tw:Play()
	return tw
end

local function logoText(c)
	return string.format('<font color="#%s">C</font><font color="#FFFFFF">B</font>', c:ToHex())
end

local function fmt(n) -- 12345 -> "12.345"
	local s = tostring(math.floor(n))
	local out = s:reverse():gsub("(%d%d%d)", "%1."):reverse()
	if out:sub(1, 1) == "." then out = out:sub(2) end
	return out
end

local Linear = Enum.EasingStyle.Linear
local In     = Enum.EasingDirection.In

--------------------------------------------------------------------
-- BIBLIOTECA
--------------------------------------------------------------------
local Library = {}

function Library:CreateWindow(config)
	config = config or {}
	if config.Accent then Theme.Accent = config.Accent end

	local VERSION = config.Version or "v1.0"
	local W, H = 660, 420
	local SIDE = 150

	local Window = {
		CurrentTab = nil,
		ToggleKey = config.ToggleKey or Enum.KeyCode.RightShift,
		ButtonEnabled = true,
		Settings = {
			Volume = config.Volume or 0.7,
			Pitch  = config.Pitch or 1,
			Muted  = false,
		},
	}

	local old = LocalPlayer.PlayerGui:FindFirstChild("ControlBlox")
	if old then old:Destroy() end

	local gui = new("ScreenGui", {
		Name = "ControlBlox",
		ResetOnSpawn = false,
		IgnoreGuiInset = true,
		ZIndexBehavior = Enum.ZIndexBehavior.Sibling,
		DisplayOrder = 999,
		Parent = LocalPlayer:WaitForChild("PlayerGui"),
	})

	----------------------------------------------------------------
	-- COR DE DESTAQUE (registro de objetos que mudam de cor)
	----------------------------------------------------------------
	local accentFns = {}
	local function onAccent(fn)
		table.insert(accentFns, fn)
		fn(Theme.Accent)
	end

    local currentDesign = "Original"

    local function applyDesignToObject(obj, preset)
        if not obj or not obj:IsA("GuiObject") then return end
        if obj:IsA("TextButton") then
            obj.BackgroundColor3 = preset.Surface
            obj.TextColor3 = preset.Text
        elseif obj:IsA("TextBox") then
            obj.BackgroundColor3 = preset.Bg
            obj.TextColor3 = preset.Text
            obj.PlaceholderColor3 = preset.Sub
        elseif obj:IsA("TextLabel") then
            if obj.TextColor3 == Theme.Accent then obj.TextColor3 = preset.Accent
            elseif obj.TextColor3 == Theme.Sub then obj.TextColor3 = preset.Sub
            elseif obj.TextColor3 ~= Theme.Good and obj.TextColor3 ~= Theme.Warn and obj.TextColor3 ~= Theme.Bad then obj.TextColor3 = preset.Text end
        elseif obj:IsA("ScrollingFrame") then
            obj.ScrollBarImageColor3 = preset.Line
        elseif obj:IsA("Frame") or obj:IsA("CanvasGroup") then
            if obj.BackgroundTransparency < 1 then
                local old = obj.BackgroundColor3
                if old == Theme.Bg or old == Theme.Black then obj.BackgroundColor3 = preset.Bg
                elseif old == Theme.Hover then obj.BackgroundColor3 = preset.Hover
                elseif old == Theme.Accent then obj.BackgroundColor3 = preset.Accent
                elseif old == Theme.Line then obj.BackgroundColor3 = preset.Line
                else obj.BackgroundColor3 = preset.Surface end
            end
        end
        for _, child in ipairs(obj:GetChildren()) do
            if child:IsA("UICorner") then
                -- Mantém os cantos arredondados definidos pelo componente.
            elseif child:IsA("UIStroke") then
                child.Color = preset.Line
            end
        end
    end

    function Window:SetDesign(name)
        local preset = Designs[name]
        if not preset then
            warn("Control Blox: design desconhecido '" .. tostring(name) .. "'.")
            return false
        end
        currentDesign = name
        -- Atualiza os objetos usando a paleta anterior para identificar suas cores atuais.
        for _, obj in ipairs(gui:GetDescendants()) do
            applyDesignToObject(obj, preset)
        end
        for key, value in pairs(preset) do Theme[key] = value end
        for _, fn in ipairs(accentFns) do pcall(fn, Theme.Accent) end
        -- Reaplica a cor de destaque aos elementos registrados dinamicamente.
        self:Notify("Design aplicado", name, 2)
        return true
    end

    function Window:GetDesign()
        return currentDesign
    end

    function Window:ApplyDesign(object, name)
        local preset = Designs[name or currentDesign]
        if not preset then return false end
        applyDesignToObject(object, preset)
        return true
    end

	function Window:SetAccent(c)
		Theme.Accent = c
		for _, fn in ipairs(accentFns) do pcall(fn, c) end
	end

	----------------------------------------------------------------
	-- SONS
	----------------------------------------------------------------
	local soundObjs = {}
	for name, info in pairs(Sounds) do
		soundObjs[name] = new("Sound", { Name = name, SoundId = info.id, Parent = gui })
	end

	local function play(name, extraSpeed)
		if Window.Settings.Muted then return end
		local info, s = Sounds[name], soundObjs[name]
		if not (info and s) then return end
		s.Volume = info.vol * Window.Settings.Volume
		s.PlaybackSpeed = math.max(0.1, info.speed * Window.Settings.Pitch * (extraSpeed or 1))
		s.TimePosition = 0
		s:Play()
	end

	function Window:PlaySound(name, speed) play(name, speed) end
	function Window:SetVolume(v) Window.Settings.Volume = math.clamp(v, 0, 1) end
	function Window:SetPitch(p) Window.Settings.Pitch = math.clamp(p, 0.5, 2) end

	----------------------------------------------------------------
	-- JANELA
	----------------------------------------------------------------
	local isOpen, collapsed, ready = false, false, false

	local root = new("Frame", {
		Size = UDim2.fromOffset(W, H),
		Position = UDim2.fromScale(0.5, 0.5),
		AnchorPoint = Vector2.new(0.5, 0.5),
		BackgroundTransparency = 1,
		Visible = false,
		Parent = gui,
	})
	local scale = new("UIScale", { Scale = 0.94, Parent = root })

	local main = new("CanvasGroup", {
		Size = UDim2.fromScale(1, 1),
		BackgroundColor3 = Theme.Bg,
		BorderSizePixel = 0,
		GroupTransparency = 1,
		ClipsDescendants = true, -- ADICIONE ESTA LINHA
		Parent = root,
	}, { corner(10), stroke(Theme.Line, 1) })

	-- topbar
	local topbar = new("Frame", {
		Size = UDim2.new(1, 0, 0, 44),
		BackgroundTransparency = 1,
		Parent = main,
	})

	local topLogo = new("TextLabel", {
		Size = UDim2.fromOffset(40, 44),
		Position = UDim2.fromOffset(16, 0),
		BackgroundTransparency = 1,
		RichText = true,
		Text = logoText(Theme.Accent),
		TextSize = 20,
		Font = Theme.Title,
		TextXAlignment = Enum.TextXAlignment.Left,
		Parent = topbar,
	})
	onAccent(function(c) topLogo.Text = logoText(c) end)

	new("TextLabel", {
		Size = UDim2.new(0, 200, 1, 0),
		Position = UDim2.fromOffset(54, 0),
		BackgroundTransparency = 1,
		Text = "Control Blox  " .. VERSION,
		TextColor3 = Theme.Sub,
		TextSize = 12,
		Font = Theme.Body,
		TextXAlignment = Enum.TextXAlignment.Left,
		Parent = topbar,
	})

	local function topButton(text, x)
		local b = new("TextButton", {
			Size = UDim2.fromOffset(28, 28),
			Position = UDim2.new(1, x, 0.5, -14),
			BackgroundTransparency = 1,
			Text = text,
			TextColor3 = Theme.Sub,
			TextSize = 16,
			Font = Theme.Body,
			AutoButtonColor = false,
			Parent = topbar,
		})
		b.MouseEnter:Connect(function() tween(b, 0.15, { TextColor3 = Theme.Text }) end)
		b.MouseLeave:Connect(function() tween(b, 0.15, { TextColor3 = Theme.Sub }) end)
		return b
	end
	local closeBtn = topButton("×", -38)
	local minBtn   = topButton("–", -66)

	-- linha de detalhes (cor de destaque)
	local accentLine = new("Frame", {
		Size = UDim2.new(1, 0, 0, 1),
		Position = UDim2.fromOffset(0, 44),
		BackgroundColor3 = Theme.Surface,
		BorderSizePixel = 0,
		Parent = main,
	})
	onAccent(function(c) tween(accentLine, 0.25, { BackgroundColor3 = c }) end)

	-- sidebar
	local tabList = new("Frame", {
		Size = UDim2.new(0, SIDE, 1, -45 - 72),
		Position = UDim2.fromOffset(0, 45),
		BackgroundTransparency = 1,
		Parent = main,
	}, {
		new("UIListLayout", { Padding = UDim.new(0, 2), SortOrder = Enum.SortOrder.LayoutOrder }),
		new("UIPadding", { PaddingTop = UDim.new(0, 12), PaddingLeft = UDim.new(0, 10), PaddingRight = UDim.new(0, 10) }),
	})

	new("Frame", {
		Size = UDim2.new(0, 1, 1, -45),
		Position = UDim2.fromOffset(SIDE, 45),
		BackgroundColor3 = Theme.Line,
		BorderSizePixel = 0,
		Parent = main,
	})

	local content = new("Frame", {
		Size = UDim2.new(1, -SIDE - 1, 1, -45),
		Position = UDim2.fromOffset(SIDE + 1, 45),
		BackgroundTransparency = 1,
		ClipsDescendants = true,
		Parent = main,
	})

	----------------------------------------------------------------
	-- DADOS DO PERFIL (mostrados na aba Perfil) + cartão pequeno na lateral
	----------------------------------------------------------------
	local State = {
		fps = "—", fpsColor = Theme.Text,
		ping = "—",
		avatar = "calculando", avatarColor = Theme.Sub,
		body = "",
	}
	local profileUI = {}

	local function setStat(key, text, color)
		State[key] = text
		if color then State[key .. "Color"] = color end
		local lbl = profileUI[key]
		if lbl then
			lbl.Text = text
			if color then lbl.TextColor3 = color end
		end
	end

	-- cartão pequeno (avatarzinho + nome, embaixo na lateral)
	local profile = new("Frame", {
		Size = UDim2.fromOffset(SIDE - 20, 52),
		Position = UDim2.new(0, 10, 1, -62),
		BackgroundColor3 = Theme.Surface,
		Parent = main,
	}, { corner(8), stroke(Theme.Line, 1) })

	local avatar = new("ImageLabel", {
		Size = UDim2.fromOffset(34, 34),
		Position = UDim2.fromOffset(9, 9),
		BackgroundColor3 = Theme.Bg,
		Image = "",
		Parent = profile,
	}, { new("UICorner", { CornerRadius = UDim.new(1, 0) }) })

	task.spawn(function()
		local ok, img = pcall(function()
			return Players:GetUserThumbnailAsync(LocalPlayer.UserId, Enum.ThumbnailType.HeadShot, Enum.ThumbnailSize.Size100x100)
		end)
		if ok then avatar.Image = img end
	end)

	new("TextLabel", {
		Size = UDim2.fromOffset(SIDE - 20 - 56, 16),
		Position = UDim2.fromOffset(50, 9),
		BackgroundTransparency = 1,
		Text = LocalPlayer.DisplayName,
		TextColor3 = Theme.Text,
		TextSize = 12,
		Font = Theme.Bold,
		TextXAlignment = Enum.TextXAlignment.Left,
		TextTruncate = Enum.TextTruncate.AtEnd,
		Parent = profile,
	})
	new("TextLabel", {
		Size = UDim2.fromOffset(SIDE - 20 - 56, 14),
		Position = UDim2.fromOffset(50, 26),
		BackgroundTransparency = 1,
		Text = "@" .. LocalPlayer.Name,
		TextColor3 = Theme.Sub,
		TextSize = 10,
		Font = Theme.Body,
		TextXAlignment = Enum.TextXAlignment.Left,
		TextTruncate = Enum.TextTruncate.AtEnd,
		Parent = profile,
	})

	-- avatar inteiro (igual ao perfil do Roblox)
	local function loadFullBody()
		task.spawn(function()
			for _ = 1, 6 do
				local ok, img, isReady = pcall(function()
					return Players:GetUserThumbnailAsync(LocalPlayer.UserId, Enum.ThumbnailType.AvatarThumbnail, Enum.ThumbnailSize.Size420x420)
				end)
				if ok and img then
					State.body = img
					if profileUI.body then profileUI.body.Image = img end
					if isReady then return end
				end
				task.wait(1.5)
			end
		end)
	end

	-- FPS e ping
	do
		local frames, acc = 0, 0
		RunService.RenderStepped:Connect(function(dt)
			frames += 1
			acc += dt
			if acc < 0.5 then return end
			local fps = math.floor(frames / acc + 0.5)
			frames, acc = 0, 0
			if not isOpen then return end
			setStat("fps", tostring(fps), fps >= 50 and Theme.Good or (fps >= 30 and Theme.Warn or Theme.Bad))
			local ok, ping = pcall(function() return Stats.Network.ServerStatsItem["Data Ping"]:GetValue() end)
			if ok and ping then setStat("ping", math.floor(ping) .. " ms") end
		end)
	end

	-- valor do avatar em Robux (soma dos itens vestidos que têm preço)
	do
		local computing = false

		local function getDescription()
			local char = LocalPlayer.Character
			local hum = char and char:FindFirstChildOfClass("Humanoid")
			if hum then
				local ok, d = pcall(function() return hum:GetAppliedDescription() end)
				if ok and d then return d end
			end
			local ok, d = pcall(function() return Players:GetHumanoidDescriptionFromUserId(LocalPlayer.UserId) end)
			if ok then return d end
			return nil
		end

		local function collectIds(d)
			local ids, seen = {}, {}
			local function add(id)
				if type(id) == "number" and id > 0 and not seen[id] then
					seen[id] = true
					table.insert(ids, id)
				end
			end
			for _, a in ipairs(d:GetAccessories(true)) do add(a.AssetId) end
			for _, prop in ipairs({ "Shirt", "Pants", "GraphicTShirt", "Face", "Head", "Torso", "LeftArm", "RightArm", "LeftLeg", "RightLeg" }) do
				add(d[prop])
			end
			return ids
		end

		local function refresh()
			if computing then return end
			computing = true
			setStat("avatar", "calculando", Theme.Sub)
			task.spawn(function()
				local d = getDescription()
				if not d then
					setStat("avatar", "indisponível", Theme.Sub)
					computing = false
					return
				end
				local okIds, ids = pcall(collectIds, d)
				if not okIds then
					setStat("avatar", "indisponível", Theme.Sub)
					computing = false
					return
				end
				local total = 0
				for _, id in ipairs(ids) do
					local ok, info = pcall(function() return MarketplaceService:GetProductInfo(id, Enum.InfoType.Asset) end)
					if ok and info and type(info.PriceInRobux) == "number" then
						total += info.PriceInRobux
						setStat("avatar", "R$ " .. fmt(total), Theme.Text)
					end
				end
				setStat("avatar", "R$ " .. fmt(total), Theme.Text)
				computing = false
			end)
		end

		Window.RefreshProfile = function()
			loadFullBody()
			refresh()
		end

		task.spawn(function()
			if not LocalPlayer.Character then LocalPlayer.CharacterAdded:Wait() end
			task.wait(1)
			loadFullBody()
			refresh()
		end)
		LocalPlayer.CharacterAdded:Connect(function()
			task.wait(2)
			loadFullBody()
			refresh()
		end)
	end

	----------------------------------------------------------------
	-- ARRASTAR JANELA
	----------------------------------------------------------------
	do
		local dragging, dragStart, startPos
		topbar.InputBegan:Connect(function(input)
			if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
				dragging, dragStart, startPos = true, input.Position, root.Position
			end
		end)
		UserInputService.InputChanged:Connect(function(input)
			if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
				local d = input.Position - dragStart
				root.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + d.X, startPos.Y.Scale, startPos.Y.Offset + d.Y)
			end
		end)
		UserInputService.InputEnded:Connect(function(input)
			if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
				dragging = false
			end
		end)
	end

	----------------------------------------------------------------
	-- NOTIFICAÇÕES
	----------------------------------------------------------------
	local notifHolder = new("Frame", {
		Size = UDim2.fromOffset(280, 400),
		Position = UDim2.new(1, -20, 1, -20),
		AnchorPoint = Vector2.new(1, 1),
		BackgroundTransparency = 1,
		ZIndex = 50,
		Parent = gui,
	}, {
		new("UIListLayout", {
			Padding = UDim.new(0, 6),
			VerticalAlignment = Enum.VerticalAlignment.Bottom,
			SortOrder = Enum.SortOrder.LayoutOrder,
		}),
	})

	function Window:Notify(title, text, duration)
		duration = duration or 4
		play("notify")

		local holder = new("Frame", {
			Size = UDim2.new(1, 0, 0, 58),
			BackgroundTransparency = 1,
			Parent = notifHolder,
		})
		local card = new("Frame", {
			Size = UDim2.fromScale(1, 1),
			Position = UDim2.new(1, 300, 0, 0),
			BackgroundColor3 = Theme.Bg,
			ClipsDescendants = true,
			Parent = holder,
		}, { corner(8), stroke(Theme.Line, 1) })

		new("Frame", {
			Size = UDim2.new(0, 2, 0, 26),
			Position = UDim2.fromOffset(12, 16),
			BackgroundColor3 = Theme.Surface,
			BorderSizePixel = 0,
			Parent = card,
		})
		new("TextLabel", {
			Size = UDim2.new(1, -36, 0, 18),
			Position = UDim2.fromOffset(24, 10),
			BackgroundTransparency = 1,
			Text = title,
			TextColor3 = Theme.Text,
			TextSize = 13,
			Font = Theme.Bold,
			TextXAlignment = Enum.TextXAlignment.Left,
			Parent = card,
		})
		new("TextLabel", {
			Size = UDim2.new(1, -36, 0, 16),
			Position = UDim2.fromOffset(24, 29),
			BackgroundTransparency = 1,
			Text = text or "",
			TextColor3 = Theme.Sub,
			TextSize = 11,
			Font = Theme.Body,
			TextXAlignment = Enum.TextXAlignment.Left,
			TextTruncate = Enum.TextTruncate.AtEnd,
			Parent = card,
		})
		local bar = new("Frame", {
			Size = UDim2.new(1, 0, 0, 1),
			Position = UDim2.new(0, 0, 1, -1),
			BackgroundColor3 = Theme.Accent,
			BorderSizePixel = 0,
			Parent = card,
		})

		tween(card, 0.5, { Position = UDim2.fromOffset(0, 0) })
		tween(bar, duration, { Size = UDim2.new(0, 0, 0, 1) }, Linear)

		task.delay(duration, function()
			tween(card, 0.35, { Position = UDim2.new(1, 300, 0, 0) }, Enum.EasingStyle.Quint, In)
			task.wait(0.36)
			tween(holder, 0.2, { Size = UDim2.new(1, 0, 0, 0) })
			task.wait(0.22)
			holder:Destroy()
		end)
	end

	----------------------------------------------------------------
	-- BOTÃO FLUTUANTE (abrir / fechar) — sempre visível e arrastável
	----------------------------------------------------------------
	local fab = new("TextButton", {
		Name = "ToggleButton",
		Size = UDim2.fromOffset(44, 44),
		AnchorPoint = Vector2.new(0.5, 0.5),
		Position = UDim2.new(0, 40, 0.5, 0),
		BackgroundColor3 = Theme.Bg,
		RichText = true,
		Text = logoText(Theme.Accent),
		TextSize = 17,
		Font = Theme.Title,
		AutoButtonColor = false,
		Visible = false,
		ZIndex = 60,
		Parent = gui,
	}, { new("UICorner", { CornerRadius = UDim.new(1, 0) }) })
	local fabStroke = stroke(Theme.Line, 1.5)
	fabStroke.Parent = fab

	local function updateFab()
		fab.Visible = ready and Window.ButtonEnabled
		tween(fabStroke, 0.25, { Color = isOpen and Theme.Accent or Theme.Line })
	end
	onAccent(function(c)
		fab.Text = logoText(c)
		if isOpen then tween(fabStroke, 0.25, { Color = c }) end
	end)

	function Window:SetButtonVisible(v)
		Window.ButtonEnabled = v and true or false
		updateFab()
	end

	local function setOpen(v)
		if v == isOpen or not ready then return end
		isOpen = v
		play(v and "open" or "close")
		if v then
			root.Visible = true
			scale.Scale = 0.94
			main.GroupTransparency = 1
			tween(scale, 0.4, { Scale = 1 })
			tween(main, 0.3, { GroupTransparency = 0 })
		else
			tween(scale, 0.25, { Scale = 0.96 }, Enum.EasingStyle.Quint, In)
			tween(main, 0.25, { GroupTransparency = 1 })
			task.delay(0.26, function()
				if not isOpen then root.Visible = false end
			end)
		end
		updateFab()
	end

	function Window:Toggle() setOpen(not isOpen) end

	-- arrastar o botão (clique curto = abrir/fechar)
	do
		local down, moved, startPos, startMouse = false, false, nil, nil
		fab.InputBegan:Connect(function(input)
			if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
				down, moved = true, false
				startMouse, startPos = input.Position, fab.Position
			end
		end)
		UserInputService.InputChanged:Connect(function(input)
			if down and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
				local d = input.Position - startMouse
				if d.Magnitude > 6 then moved = true end
				if moved then
					fab.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + d.X, startPos.Y.Scale, startPos.Y.Offset + d.Y)
				end
			end
		end)
		UserInputService.InputEnded:Connect(function(input)
			if down and (input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch) then
				down = false
				if not moved then setOpen(not isOpen) end
			end
		end)
		fab.MouseEnter:Connect(function() tween(fab, 0.2, { Size = UDim2.fromOffset(50, 50) }) end)
		fab.MouseLeave:Connect(function() tween(fab, 0.2, { Size = UDim2.fromOffset(44, 44) }) end)
	end




minBtn.MouseButton1Click:Connect(function()
    play("click")
    collapsed = not collapsed

    if collapsed then
        -- Mantém apenas a barra superior visível
        for _, obj in ipairs(main:GetChildren()) do
            if obj ~= topbar and obj:IsA("GuiObject") then
                obj.Visible = false
            end
        end

        tween(root, 0.25, {
            Size = UDim2.fromOffset(W, 45)
        })
    else
        -- Restaura o tamanho da janela
        tween(root, 0.25, {
            Size = UDim2.fromOffset(W, H)
        })

        -- Mostra novamente os elementos da interface
        for _, obj in ipairs(main:GetChildren()) do
            if obj:IsA("GuiObject") then
                obj.Visible = true
            end
        end
    end

    -- Garante que os botões continuem disponíveis
    minBtn.Visible = true
    closeBtn.Visible = true
end)




	----------------------------------------------------------------
	-- INTRODUÇÃO
	----------------------------------------------------------------
	local function playIntro()
		local abs = gui.AbsoluteSize
		local cx, cy = math.floor(abs.X / 2), math.floor(abs.Y / 2)
		local accentHex = Theme.Accent:ToHex()

		local intro = new("Frame", {
			Size = UDim2.fromScale(1, 1),
			BackgroundColor3 = Theme.Black,
			BorderSizePixel = 0,
			ZIndex = 100,
			Parent = gui,
		})

		local function label(props)
			props.BackgroundTransparency = 1
			props.Parent = intro
			return new("TextLabel", props)
		end

		local logoY = cy - 110
		local logo = label({
			Size = UDim2.fromOffset(200, 80),
			Position = UDim2.fromOffset(cx - 100, logoY - 10),
			RichText = true,
			Text = logoText(Theme.Accent),
			TextSize = 72,
			Font = Theme.Title,
			TextTransparency = 1,
		})

		local line = new("Frame", {
			Size = UDim2.fromOffset(0, 1),
			Position = UDim2.fromOffset(cx, cy - 16),
			BackgroundColor3 = Theme.Line,
			BorderSizePixel = 0,
			Parent = intro,
		})

		local title = label({
			Size = UDim2.fromOffset(320, 20),
			Position = UDim2.fromOffset(cx - 160, cy + 2),
			Text = "C O N T R O L    B L O X",
			TextColor3 = Theme.Text,
			TextTransparency = 1,
			TextSize = 14,
			Font = Theme.Bold,
		})

		local block = new("Frame", {
			Size = UDim2.fromOffset(240, 34),
			Position = UDim2.fromOffset(cx - 120, cy + 52),
			BackgroundTransparency = 1,
			Parent = intro,
		})
		local status = new("TextLabel", {
			Size = UDim2.fromOffset(168, 14),
			BackgroundTransparency = 1,
			Text = "iniciando",
			TextColor3 = Theme.Sub,
			TextTransparency = 1,
			TextSize = 12,
			Font = Theme.Mono,
			TextXAlignment = Enum.TextXAlignment.Left,
			Parent = block,
		})
		local pct = new("TextLabel", {
			Size = UDim2.fromOffset(72, 14),
			Position = UDim2.fromOffset(168, 0),
			BackgroundTransparency = 1,
			Text = "000%",
			TextColor3 = Theme.Text,
			TextTransparency = 1,
			TextSize = 12,
			Font = Theme.Mono,
			TextXAlignment = Enum.TextXAlignment.Right,
			Parent = block,
		})
		local track = new("Frame", {
			Size = UDim2.fromOffset(240, 1),
			Position = UDim2.fromOffset(0, 24),
			BackgroundColor3 = Theme.Line,
			BackgroundTransparency = 1,
			BorderSizePixel = 0,
			Parent = block,
		})
		local fill = new("Frame", {
			Size = UDim2.fromScale(0, 1),
			BackgroundColor3 = Theme.Surface,
			BorderSizePixel = 0,
			Parent = track,
		})

		local log = new("Frame", {
			Size = UDim2.fromOffset(260, 80),
			Position = UDim2.fromOffset(28, abs.Y - 28 - 80),
			BackgroundTransparency = 1,
			Parent = intro,
		}, {
			new("UIListLayout", { VerticalAlignment = Enum.VerticalAlignment.Bottom, Padding = UDim.new(0, 2) }),
		})

		local function addLog(text)
			local l = new("TextLabel", {
				Size = UDim2.new(1, 0, 0, 14),
				BackgroundTransparency = 1,
				RichText = true,
				Text = '<font color="#' .. accentHex .. '">›</font> ' .. text,
				TextColor3 = Theme.Sub,
				TextTransparency = 1,
				TextSize = 12,
				Font = Theme.Mono,
				TextXAlignment = Enum.TextXAlignment.Left,
				Parent = log,
			})
			tween(l, 0.4, { TextTransparency = 0 })
		end

		local ver = label({
			Size = UDim2.fromOffset(200, 14),
			Position = UDim2.fromOffset(abs.X - 28 - 200, abs.Y - 28 - 14),
			Text = VERSION .. "  •  " .. LocalPlayer.Name,
			TextColor3 = Theme.Sub,
			TextTransparency = 1,
			TextSize = 12,
			Font = Theme.Mono,
			TextXAlignment = Enum.TextXAlignment.Right,
		})

		local nv = new("NumberValue", { Value = 0, Parent = intro })
		nv.Changed:Connect(function(v)
			fill.Size = UDim2.fromScale(v, 1)
			pct.Text = string.format("%03d%%", math.floor(v * 100 + 0.5))
		end)

		task.wait(0.3)
		tween(logo, 0.9, { TextTransparency = 0, Position = UDim2.fromOffset(cx - 100, logoY) })
		task.wait(0.6)

		tween(line, 0.8, { Size = UDim2.fromOffset(240, 1), Position = UDim2.fromOffset(cx - 120, cy - 16) })
		task.wait(0.4)
		tween(title, 0.8, { TextTransparency = 0 })
		task.wait(0.4)

		tween(status, 0.4, { TextTransparency = 0 })
		tween(pct, 0.4, { TextTransparency = 0 })
		tween(track, 0.4, { BackgroundTransparency = 0 })
		tween(ver, 0.6, { TextTransparency = 0 })
		task.wait(0.4)

		local steps = {
			{ "carregando interface", 0.25, "interface .......... ok" },
			{ "carregando módulos",   0.52, "módulos ............ ok" },
			{ "aplicando tema",       0.78, "tema ............... ok" },
			{ "verificando jogador",  0.95, "jogador ............ ok" },
			{ "pronto",               1.00, "control blox ....... pronto" },
		}
		for _, s in ipairs(steps) do
			status.Text = s[1]
			tween(nv, 0.55, { Value = s[2] }, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut)
			task.wait(0.4)
			addLog(s[3])
			play("slider")
			task.wait(0.25)
		end
		task.wait(0.5)

		tween(line, 0.5, { Size = UDim2.fromOffset(0, 1), Position = UDim2.fromOffset(cx, cy - 16) }, Enum.EasingStyle.Quint, In)
		tween(logo, 0.7, { Position = UDim2.fromOffset(cx - 100, logoY - 10) })
		task.wait(0.35)

		for _, d in ipairs(intro:GetDescendants()) do
			if d:IsA("TextLabel") then
				tween(d, 0.6, { TextTransparency = 1 })
			elseif d:IsA("Frame") then
				tween(d, 0.6, { BackgroundTransparency = 1 })
			end
		end
		tween(intro, 0.7, { BackgroundTransparency = 1 })
		task.wait(0.75)
		intro:Destroy()
	end

	task.spawn(function()
		if config.Intro ~= false then playIntro() end
		ready = true
		setOpen(true)
		updateFab()
		task.wait(0.5)
		Window:Notify("CONTROL BLOX", "Menu iniciado  •  " .. Window.ToggleKey.Name .. " para ocultar", 5)
	end)

	----------------------------------------------------------------
	-- ABAS
	----------------------------------------------------------------
	function Window:AddTab(name)
		local Tab = {}

	local tabOrder = {
		["Início"] = 1,
		["Jogador"] = 2,
		["Exploits"] = 3,
		["Perfil"] = 4,
		["Config"] = 5,
	}

	local btn = new("TextButton", {
		Size = UDim2.new(1, 0, 0, 32),
		LayoutOrder = tabOrder[name] or 0,
		BackgroundColor3 = Theme.Surface,
		BackgroundTransparency = 1,
		Text = "",
		AutoButtonColor = false,
		Parent = tabList,
	}, { corner(6) })

		local dot = new("Frame", {
			Size = UDim2.fromOffset(2, 0),
			Position = UDim2.new(0, 0, 0.5, 0),
			AnchorPoint = Vector2.new(0, 0.5),
			BackgroundColor3 = Theme.Accent,
			BorderSizePixel = 0,
			Parent = btn,
		})
		onAccent(function(c) dot.BackgroundColor3 = c end)

		local btnLabel = new("TextLabel", {
			Size = UDim2.new(1, -20, 1, 0),
			Position = UDim2.fromOffset(12, 0),
			BackgroundTransparency = 1,
			Text = name,
			TextColor3 = Theme.Sub,
			TextSize = 13,
			Font = Theme.Body,
			TextXAlignment = Enum.TextXAlignment.Left,
			Parent = btn,
		})

		local page = new("ScrollingFrame", {
			Size = UDim2.fromScale(1, 1),
			BackgroundTransparency = 1,
			BorderSizePixel = 0,
			ScrollBarThickness = 2,
			ScrollBarImageColor3 = Theme.Line,
			AutomaticCanvasSize = Enum.AutomaticSize.Y,
			CanvasSize = UDim2.new(),
			Visible = false,
			Parent = content,
		}, {
			new("UIListLayout", { Padding = UDim.new(0, 6), SortOrder = Enum.SortOrder.LayoutOrder }),
			new("UIPadding", {
				PaddingTop = UDim.new(0, 14), PaddingLeft = UDim.new(0, 14),
				PaddingRight = UDim.new(0, 16), PaddingBottom = UDim.new(0, 14),
			}),
		})

		local entry = { page = page, btn = btn, label = btnLabel, dot = dot }
		Tab.Page = page

		local function select(silent)
			if Window.CurrentTab == entry then return end
			local prev = Window.CurrentTab
			if prev then
				prev.page.Visible = false
				tween(prev.btn, 0.2, { BackgroundTransparency = 1 })
				tween(prev.label, 0.2, { TextColor3 = Theme.Sub })
				tween(prev.dot, 0.2, { Size = UDim2.fromOffset(2, 0) })
			end
			Window.CurrentTab = entry
			if not silent then play("tab") end
			page.Position = UDim2.fromOffset(0, 10)
			page.Visible = true
			tween(page, 0.35, { Position = UDim2.fromOffset(0, 0) })
			tween(btn, 0.25, { BackgroundTransparency = 0 })
			tween(btnLabel, 0.25, { TextColor3 = Theme.Text })
			tween(dot, 0.3, { Size = UDim2.fromOffset(2, 14) })
		end

		btn.MouseButton1Click:Connect(function() select(false) end)
		btn.MouseEnter:Connect(function()
			if Window.CurrentTab ~= entry then tween(btnLabel, 0.15, { TextColor3 = Theme.Text }) end
		end)
		btn.MouseLeave:Connect(function()
			if Window.CurrentTab ~= entry then tween(btnLabel, 0.15, { TextColor3 = Theme.Sub }) end
		end)
		if not Window.CurrentTab then select(true) end

		----------------------------------------------------------
		-- bases
		----------------------------------------------------------
		local function element(height)
			local f = new("Frame", {
				Size = UDim2.new(1, 0, 0, height or 36),
				BackgroundColor3 = Theme.Surface,
				BorderSizePixel = 0,
				ClipsDescendants = true,
				Parent = page,
			}, { corner(7) })
			f.MouseEnter:Connect(function() tween(f, 0.15, { BackgroundColor3 = Theme.Hover }) end)
			f.MouseLeave:Connect(function() tween(f, 0.15, { BackgroundColor3 = Theme.Surface }) end)
			return f
		end

		local function label(parent, text, size)
			return new("TextLabel", {
				Size = size or UDim2.new(1, -70, 0, 36),
				Position = UDim2.fromOffset(12, 0),
				BackgroundTransparency = 1,
				Text = text,
				TextColor3 = Theme.Text,
				TextSize = 13,
				Font = Theme.Body,
				TextXAlignment = Enum.TextXAlignment.Left,
				Parent = parent,
			})
		end

		----------------------------------------------------------
		function Tab:AddSection(text)
			new("TextLabel", {
				Size = UDim2.new(1, 0, 0, 24),
				BackgroundTransparency = 1,
				Text = string.upper(text),
				TextColor3 = Theme.Sub,
				TextSize = 10,
				Font = Theme.Bold,
				TextXAlignment = Enum.TextXAlignment.Left,
				Parent = page,
			})
		end

		function Tab:AddLabel(text)
			local f = element(30)
			f.BackgroundTransparency = 1
			local l = label(f, text, UDim2.new(1, -24, 1, 0))
			l.TextColor3 = Theme.Sub
			l.TextSize = 12
			l.Position = UDim2.fromOffset(2, 0)
			return { Set = function(_, t) l.Text = t end }
		end

		----------------------------------------------------------
		function Tab:AddButton(text, callback)
			local f = element(36)
			label(f, text)
			local arrow = new("TextLabel", {
				Size = UDim2.fromOffset(24, 36),
				Position = UDim2.new(1, -30, 0, 0),
				BackgroundTransparency = 1,
				Text = "→",
				TextColor3 = Theme.Sub,
				TextSize = 14,
				Font = Theme.Body,
				Parent = f,
			})
			local b = new("TextButton", { Size = UDim2.fromScale(1, 1), BackgroundTransparency = 1, Text = "", Parent = f })
			b.MouseEnter:Connect(function()
				tween(arrow, 0.2, { Position = UDim2.new(1, -24, 0, 0), TextColor3 = Theme.Accent })
			end)
			b.MouseLeave:Connect(function()
				tween(arrow, 0.2, { Position = UDim2.new(1, -30, 0, 0), TextColor3 = Theme.Sub })
			end)
			b.MouseButton1Click:Connect(function()
				play("click")
				if callback then task.spawn(callback) end
			end)
		end

		----------------------------------------------------------
		function Tab:AddToggle(text, default, callback)
			local state = default or false
			local f = element(36)
			label(f, text)

			local track = new("Frame", {
				Size = UDim2.fromOffset(34, 18),
				Position = UDim2.new(1, -46, 0.5, -9),
				BackgroundColor3 = state and Theme.Accent or Theme.Line,
				Parent = f,
			}, { new("UICorner", { CornerRadius = UDim.new(1, 0) }) })
			local knob = new("Frame", {
				Size = UDim2.fromOffset(12, 12),
				Position = state and UDim2.fromOffset(19, 3) or UDim2.fromOffset(3, 3),
				BackgroundColor3 = Theme.Text,
				Parent = track,
			}, { new("UICorner", { CornerRadius = UDim.new(1, 0) }) })

			onAccent(function(c) if state then track.BackgroundColor3 = c end end)

			local function set(v)
				state = v
				play("toggle", state and 1.15 or 0.88)
				tween(track, 0.25, { BackgroundColor3 = state and Theme.Accent or Theme.Line })
				tween(knob, 0.25, { Position = state and UDim2.fromOffset(19, 3) or UDim2.fromOffset(3, 3) })
				if callback then task.spawn(callback, state) end
			end

			new("TextButton", { Size = UDim2.fromScale(1, 1), BackgroundTransparency = 1, Text = "", Parent = f })
				.MouseButton1Click:Connect(function() set(not state) end)

			if state and callback then task.spawn(callback, state) end
			return { Set = function(_, v) set(v) end, Get = function() return state end }
		end

		----------------------------------------------------------
		function Tab:AddSlider(text, min, max, default, callback)
			local value = math.clamp(default or min, min, max)
			local f = element(48)

			label(f, text, UDim2.new(1, -80, 0, 28))
			local valLabel = new("TextLabel", {
				Size = UDim2.fromOffset(60, 28),
				Position = UDim2.new(1, -72, 0, 0),
				BackgroundTransparency = 1,
				Text = tostring(value),
				TextColor3 = Theme.Sub,
				TextSize = 12,
				Font = Theme.Mono,
				TextXAlignment = Enum.TextXAlignment.Right,
				Parent = f,
			})

			local bar = new("Frame", {
				Size = UDim2.new(1, -24, 0, 3),
				Position = UDim2.fromOffset(12, 34),
				BackgroundColor3 = Theme.Line,
				BorderSizePixel = 0,
				Parent = f,
			}, { new("UICorner", { CornerRadius = UDim.new(1, 0) }) })

			local rel0 = (value - min) / (max - min)
			local fill = new("Frame", {
				Size = UDim2.fromScale(rel0, 1),
				BackgroundColor3 = Theme.Accent,
				BorderSizePixel = 0,
				Parent = bar,
			}, { new("UICorner", { CornerRadius = UDim.new(1, 0) }) })
			onAccent(function(c) fill.BackgroundColor3 = c end)

			local knob = new("Frame", {
				Size = UDim2.fromOffset(10, 10),
				Position = UDim2.new(rel0, 0, 0.5, 0),
				AnchorPoint = Vector2.new(0.5, 0.5),
				BackgroundColor3 = Theme.Text,
				Parent = bar,
			}, { new("UICorner", { CornerRadius = UDim.new(1, 0) }) })

			local function setFromX(x)
				local rel = math.clamp((x - bar.AbsolutePosition.X) / bar.AbsoluteSize.X, 0, 1)
				value = math.floor(min + (max - min) * rel + 0.5)
				local snap = (value - min) / (max - min)
				tween(fill, 0.08, { Size = UDim2.fromScale(snap, 1) })
				tween(knob, 0.08, { Position = UDim2.new(snap, 0, 0.5, 0) })
				valLabel.Text = tostring(value)
				if callback then task.spawn(callback, value) end
			end

			local dragging, conn = false, nil
			local hit = new("TextButton", {
				Size = UDim2.new(1, 0, 0, 22),
				Position = UDim2.fromOffset(0, 26),
				BackgroundTransparency = 1,
				Text = "",
				Parent = f,
			})
			hit.InputBegan:Connect(function(input)
				if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
					dragging = true
					tween(knob, 0.15, { Size = UDim2.fromOffset(14, 14) })
					tween(valLabel, 0.15, { TextColor3 = Theme.Text })
					setFromX(input.Position.X)
					conn = UserInputService.InputChanged:Connect(function(i)
						if dragging and (i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch) then
							setFromX(i.Position.X)
						end
					end)
				end
			end)
			UserInputService.InputEnded:Connect(function(input)
				if dragging and (input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch) then
					dragging = false
					if conn then conn:Disconnect() conn = nil end
					play("slider")
					tween(knob, 0.15, { Size = UDim2.fromOffset(10, 10) })
					tween(valLabel, 0.15, { TextColor3 = Theme.Sub })
				end
			end)

			return { Get = function() return value end }
		end

		----------------------------------------------------------
		function Tab:AddDropdown(text, options, default, callback)
			local current = default or options[1]
			local open = false
			local listHeight = math.min(#options * 28 + 8, 140)

			local f = element(36)
			label(f, text, UDim2.new(0.5, -14, 0, 36))
			local valLabel = new("TextLabel", {
				Size = UDim2.new(0.5, -38, 0, 36),
				Position = UDim2.fromScale(0.5, 0),
				BackgroundTransparency = 1,
				Text = tostring(current),
				TextColor3 = Theme.Sub,
				TextSize = 12,
				Font = Theme.Body,
				TextXAlignment = Enum.TextXAlignment.Right,
				Parent = f,
			})
			local arrow = new("TextLabel", {
				Size = UDim2.fromOffset(22, 36),
				Position = UDim2.new(1, -28, 0, 0),
				BackgroundTransparency = 1,
				Text = "⌄",
				TextColor3 = Theme.Sub,
				TextSize = 14,
				Font = Theme.Body,
				Parent = f,
			})

			local list = new("ScrollingFrame", {
				Size = UDim2.new(1, -16, 0, listHeight - 8),
				Position = UDim2.fromOffset(8, 40),
				BackgroundTransparency = 1,
				BorderSizePixel = 0,
				ScrollBarThickness = 2,
				ScrollBarImageColor3 = Theme.Line,
				AutomaticCanvasSize = Enum.AutomaticSize.Y,
				CanvasSize = UDim2.new(),
				Parent = f,
			}, { new("UIListLayout", { Padding = UDim.new(0, 2) }) })

			local buttons = {}
			local function refresh()
				for opt, ob in pairs(buttons) do
					tween(ob, 0.15, { TextColor3 = (opt == current) and Theme.Accent or Theme.Sub })
				end
			end
			local function toggle(v)
				open = v
				tween(f, 0.3, { Size = UDim2.new(1, 0, 0, v and (36 + listHeight) or 36) })
				tween(arrow, 0.3, { Rotation = v and 180 or 0 })
			end

			for _, opt in ipairs(options) do
				local ob = new("TextButton", {
					Size = UDim2.new(1, -4, 0, 26),
					BackgroundTransparency = 1,
					Text = tostring(opt),
					TextColor3 = Theme.Sub,
					TextSize = 12,
					Font = Theme.Body,
					TextXAlignment = Enum.TextXAlignment.Left,
					AutoButtonColor = false,
					Parent = list,
				}, { new("UIPadding", { PaddingLeft = UDim.new(0, 6) }) })
				buttons[opt] = ob
				ob.MouseEnter:Connect(function() if opt ~= current then tween(ob, 0.12, { TextColor3 = Theme.Text }) end end)
				ob.MouseLeave:Connect(function() if opt ~= current then tween(ob, 0.12, { TextColor3 = Theme.Sub }) end end)
				ob.MouseButton1Click:Connect(function()
					play("tab")
					current = opt
					valLabel.Text = tostring(opt)
					refresh()
					toggle(false)
					if callback then task.spawn(callback, opt) end
				end)
			end
			onAccent(function() refresh() end)

			new("TextButton", { Size = UDim2.new(1, 0, 0, 36), BackgroundTransparency = 1, Text = "", Parent = f })
				.MouseButton1Click:Connect(function()
					play("click")
					toggle(not open)
				end)

			return {
				Get = function() return current end,
				Set = function(_, v) current = v valLabel.Text = tostring(v) refresh() end,
			}
		end

		----------------------------------------------------------
		function Tab:AddTextBox(text, placeholder, callback)
			local f = element(36)
			label(f, text, UDim2.new(1, -190, 0, 36))
			local box = new("TextBox", {
				Size = UDim2.fromOffset(150, 24),
				Position = UDim2.new(1, -10, 0.5, 0),
				AnchorPoint = Vector2.new(1, 0.5),
				BackgroundColor3 = Theme.Bg,
				Text = "",
				PlaceholderText = placeholder or "digite...",
				PlaceholderColor3 = Theme.Sub,
				TextColor3 = Theme.Text,
				TextSize = 12,
				Font = Theme.Body,
				ClearTextOnFocus = false,
				Parent = f,
			}, { corner(5) })
			local s = stroke(Theme.Line, 1)
			s.Parent = box
			box.Focused:Connect(function() tween(s, 0.2, { Color = Theme.Accent }) end)
			box.FocusLost:Connect(function(enter)
				tween(s, 0.2, { Color = Theme.Line })
				if callback then task.spawn(callback, box.Text, enter) end
			end)
			return { Get = function() return box.Text end, Set = function(_, t) box.Text = t end }
		end

		----------------------------------------------------------
		function Tab:AddKeybind(text, default, callback)
			local key, listening = default, false
			local f = element(36)
			label(f, text)
			local kb = new("TextButton", {
				Size = UDim2.fromOffset(80, 24),
				Position = UDim2.new(1, -10, 0.5, 0),
				AnchorPoint = Vector2.new(1, 0.5),
				BackgroundColor3 = Theme.Bg,
				Text = key and key.Name or "—",
				TextColor3 = Theme.Sub,
				TextSize = 11,
				Font = Theme.Mono,
				AutoButtonColor = false,
				Parent = f,
			}, { corner(5), stroke(Theme.Line, 1) })

			kb.MouseButton1Click:Connect(function()
				play("click")
				listening = true
				kb.Text = "..."
				tween(kb, 0.15, { TextColor3 = Theme.Accent })
			end)
			UserInputService.InputBegan:Connect(function(input, processed)
				if listening and input.UserInputType == Enum.UserInputType.Keyboard then
					listening = false
					if input.KeyCode ~= Enum.KeyCode.Escape then key = input.KeyCode end
					kb.Text = key and key.Name or "—"
					tween(kb, 0.15, { TextColor3 = Theme.Sub })
				elseif not processed and key and input.KeyCode == key then
					if callback then task.spawn(callback) end
				end
			end)
			return { Get = function() return key end }
		end

		----------------------------------------------------------
		-- COLOR PICKER (cores prontas + hex personalizado)
		----------------------------------------------------------
		function Tab:AddColorPicker(text, default, callback)
			local color = default or Theme.Accent
			local open = false
			local EXPANDED = 108

			local presets = {
				"6B20FF", "2D7BFF", "00D4FF", "2EE59D",
				"FFC93C", "FF7A2F", "FF3B5C", "FF4FD8", "FFFFFF",
			}

			local f = element(36)
			label(f, text)

			local preview = new("Frame", {
				Size = UDim2.fromOffset(18, 18),
				Position = UDim2.new(1, -32, 0, 9),
				BackgroundColor3 = color,
				Parent = f,
			}, { new("UICorner", { CornerRadius = UDim.new(1, 0) }), stroke(Theme.Line, 1) })

			local row = new("Frame", {
				Size = UDim2.new(1, -24, 0, 24),
				Position = UDim2.fromOffset(12, 44),
				BackgroundTransparency = 1,
				Parent = f,
			}, { new("UIListLayout", {
				FillDirection = Enum.FillDirection.Horizontal,
				Padding = UDim.new(0, 8),
				SortOrder = Enum.SortOrder.LayoutOrder,
			}) })

			local swatches = {}
			local hexBox

			local function apply(c, silent)
				color = c
				tween(preview, 0.2, { BackgroundColor3 = c })
				for hex, sw in pairs(swatches) do
					tween(sw.stroke, 0.2, { Transparency = (hex == c:ToHex():upper()) and 0 or 1 })
				end
				if hexBox then hexBox.Text = "#" .. c:ToHex():upper() end
				if callback and not silent then task.spawn(callback, c) end
			end

			for i, hex in ipairs(presets) do
				local sw = new("TextButton", {
					Size = UDim2.fromOffset(24, 24),
					BackgroundColor3 = Color3.fromHex(hex),
					Text = "",
					AutoButtonColor = false,
					LayoutOrder = i,
					Parent = row,
				}, { new("UICorner", { CornerRadius = UDim.new(1, 0) }) })
				local st = stroke(Theme.Text, 2, 1)
				st.Parent = sw
				swatches[hex] = { btn = sw, stroke = st }
				sw.MouseButton1Click:Connect(function()
					play("tab")
					apply(Color3.fromHex(hex))
				end)
			end

			hexBox = new("TextBox", {
				Size = UDim2.fromOffset(100, 22),
				Position = UDim2.fromOffset(12, 78),
				BackgroundColor3 = Theme.Bg,
				Text = "#" .. color:ToHex():upper(),
				PlaceholderText = "#6B20FF",
				PlaceholderColor3 = Theme.Sub,
				TextColor3 = Theme.Text,
				TextSize = 12,
				Font = Theme.Mono,
				ClearTextOnFocus = false,
				Parent = f,
			}, { corner(5) })
			local hs = stroke(Theme.Line, 1)
			hs.Parent = hexBox
			hexBox.Focused:Connect(function() tween(hs, 0.2, { Color = Theme.Accent }) end)
			hexBox.FocusLost:Connect(function()
				tween(hs, 0.2, { Color = Theme.Line })
				local raw = hexBox.Text:gsub("#", ""):gsub("%s", "")
				if #raw == 6 then
					local ok, c = pcall(Color3.fromHex, raw)
					if ok and c then
						play("tab")
						apply(c)
						return
					end
				end
				hexBox.Text = "#" .. color:ToHex():upper()
			end)

			new("TextLabel", {
				Size = UDim2.fromOffset(200, 22),
				Position = UDim2.fromOffset(122, 78),
				BackgroundTransparency = 1,
				Text = "cor personalizada (hex)",
				TextColor3 = Theme.Sub,
				TextSize = 11,
				Font = Theme.Body,
				TextXAlignment = Enum.TextXAlignment.Left,
				Parent = f,
			})

			new("TextButton", { Size = UDim2.new(1, 0, 0, 36), BackgroundTransparency = 1, Text = "", Parent = f })
				.MouseButton1Click:Connect(function()
					play("click")
					open = not open
					tween(f, 0.3, { Size = UDim2.new(1, 0, 0, open and EXPANDED or 36) })
				end)

			apply(color, true)

			return {
				Get = function() return color end,
				Set = function(_, c) apply(c) end,
			}
		end

		return Tab
	end


	----------------------------------------------------------------
	-- ABA DE PERFIL (avatar inteiro + dados da conta)
	----------------------------------------------------------------
	function Window:AddProfileTab(name)
		local Tab = Window:AddTab(name or "Perfil")

		local card = new("Frame", {
			Size = UDim2.new(1, 0, 0, 264),
			BackgroundTransparency = 1,
			LayoutOrder = -1,
			Parent = Tab.Page,
		})

		-- Cartão esquerdo: avatar e botão
		local left = new("Frame", {
			Size = UDim2.fromOffset(190, 260),
			BackgroundColor3 = Theme.Surface,
			Parent = card,
		}, { corner(10) })

		-- Área do avatar
		local stage = new("Frame", {
			Size = UDim2.fromOffset(170, 170),
			Position = UDim2.fromOffset(10, 10),
			BackgroundColor3 = Theme.Bg,
			Parent = left,
		}, { corner(8) })

		-- Avatar de corpo inteiro (criado somente uma vez)
		profileUI.body = new("ImageLabel", {
			Size = UDim2.fromScale(1, 1),
			BackgroundTransparency = 1,
			Image = State.body,
			ScaleType = Enum.ScaleType.Fit,
			Parent = stage,
		})

		-- Nome de exibição
		new("TextLabel", {
			Size = UDim2.fromOffset(170, 20),
			Position = UDim2.fromOffset(10, 183),
			BackgroundTransparency = 1,
			Text = LocalPlayer.DisplayName,
			TextColor3 = Theme.Text,
			TextSize = 15,
			Font = Theme.Bold,
			TextTruncate = Enum.TextTruncate.AtEnd,
			Parent = left,
		})

		-- Nome de usuário
		new("TextLabel", {
			Size = UDim2.fromOffset(170, 14),
			Position = UDim2.fromOffset(10, 203),
			BackgroundTransparency = 1,
			Text = "@" .. LocalPlayer.Name,
			TextColor3 = Theme.Sub,
			TextSize = 11,
			Font = Theme.Body,
			TextTruncate = Enum.TextTruncate.AtEnd,
			Parent = left,
		})

		-- Botão para atualizar o avatar
		local refreshAvatar = new("TextButton", {
			Name = "RefreshAvatar",
			Size = UDim2.fromOffset(170, 28),
			Position = UDim2.fromOffset(10, 224),
			BackgroundColor3 = Theme.Accent,
			Text = "Atualizar avatar",
			TextColor3 = Theme.Text,
			TextSize = 12,
			Font = Theme.Bold,
			AutoButtonColor = false,
			Parent = left,
		}, { corner(6) })

		onAccent(function(c)
			refreshAvatar.BackgroundColor3 = c
		end)

		local updatingAvatar = false

		refreshAvatar.MouseEnter:Connect(function()
			if not updatingAvatar then
				tween(refreshAvatar, 0.15, {
					BackgroundColor3 = Theme.Accent:Lerp(Color3.new(1, 1, 1), 0.12)
				})
			end
		end)

		refreshAvatar.MouseLeave:Connect(function()
			tween(refreshAvatar, 0.15, {
				BackgroundColor3 = Theme.Accent
			})
		end)

		refreshAvatar.MouseButton1Click:Connect(function()
			if updatingAvatar then return end
			updatingAvatar = true

			refreshAvatar.Text = "Atualizando..."
			play("click")

			task.spawn(function()
				local success, err = pcall(function()
					-- Solicita novamente a miniatura do avatar
					local image = Players:GetUserThumbnailAsync(
						LocalPlayer.UserId,
						Enum.ThumbnailType.AvatarThumbnail,
						Enum.ThumbnailSize.Size420x420
					)

					State.body = image
					profileUI.body.Image = image

					-- Atualiza também o cálculo do valor dos itens
					Window:RefreshProfile()
				end)

				if not success then
					warn("[Control Blox] Erro ao atualizar avatar:", err)
					Window:Notify(
						"Perfil",
						"Não foi possível atualizar o avatar.",
						3
					)
				else
					Window:Notify(
						"Perfil",
						"Atualização solicitada!",
						2
					)
				end

				refreshAvatar.Text = "Atualizar avatar"
				updatingAvatar = false
			end)
		end)

		-- Cartões de informações à direita
		local grid = new("Frame", {
			Size = UDim2.new(1, -202, 1, 0),
			Position = UDim2.fromOffset(202, 0),
			BackgroundTransparency = 1,
			Parent = card,
		}, {
			new("UIGridLayout", {
				CellSize = UDim2.fromOffset(132, 72),
				CellPadding = UDim2.fromOffset(12, 12),
				SortOrder = Enum.SortOrder.LayoutOrder,
			}),
		})

		local function tile(order, title, text, color, key, note)
			local t = new("Frame", {
				BackgroundColor3 = Theme.Surface,
				LayoutOrder = order,
				Parent = grid,
			}, { corner(8) })

			t.MouseEnter:Connect(function()
				tween(t, 0.15, {
					BackgroundColor3 = Theme.Hover
				})
			end)

			t.MouseLeave:Connect(function()
				tween(t, 0.15, {
					BackgroundColor3 = Theme.Surface
				})
			end)

			new("TextLabel", {
				Size = UDim2.new(1, -20, 0, 12),
				Position = UDim2.fromOffset(10, 9),
				BackgroundTransparency = 1,
				Text = string.upper(title),
				TextColor3 = Theme.Sub,
				TextSize = 10,
				Font = Theme.Bold,
				TextXAlignment = Enum.TextXAlignment.Left,
				Parent = t,
			})

			local value = new("TextLabel", {
				Size = UDim2.new(1, -20, 0, 24),
				Position = UDim2.fromOffset(10, 26),
				BackgroundTransparency = 1,
				Text = text,
				TextColor3 = color or Theme.Text,
				TextScaled = true,
				Font = Theme.Bold,
				TextXAlignment = Enum.TextXAlignment.Left,
				Parent = t,
			}, {
				new("UITextSizeConstraint", {
					MaxTextSize = 18,
					MinTextSize = 8,
				}),
			})

			if note then
				new("TextLabel", {
					Size = UDim2.new(1, -20, 0, 12),
					Position = UDim2.fromOffset(10, 53),
					BackgroundTransparency = 1,
					Text = note,
					TextColor3 = Theme.Sub,
					TextSize = 9,
					Font = Theme.Body,
					TextXAlignment = Enum.TextXAlignment.Left,
					Parent = t,
				})
			end

			if key then
				profileUI[key] = value
			end

			return value
		end

		local days = LocalPlayer.AccountAge
		local premium = LocalPlayer.MembershipType == Enum.MembershipType.Premium

		tile(1, "FPS", State.fps, State.fpsColor, "fps",
			"quadros por segundo")

		tile(2, "Ping", State.ping, Theme.Text, "ping",
			"latência do servidor")

		tile(3, "Avatar", State.avatar, State.avatarColor, "avatar",
			"itens vestidos com preço")

		tile(4, "Conta", fmt(days) .. " dias", Theme.Text, nil,
			string.format("≈ %.1f anos", days / 365))

		tile(5, "Membro", premium and "Premium" or "Padrão",
			premium and Theme.Accent or Theme.Text, nil,
			"tipo de conta")

		tile(6, "ID", tostring(LocalPlayer.UserId), Theme.Text, nil,
			"UserId")

		return Tab
	end


	return Window
end

----GERAL-------

--------------------------------------------------------------------
-- EXEMPLO DE USO  (edite / apague à vontade)
--------------------------------------------------------------------
local Window = Library:CreateWindow({
	Name = "CONTROL BLOX",
	Version = "v1.0",
	ToggleKey = Enum.KeyCode.RightShift,
	Intro = true,                         -- false para pular a introdução
	Accent = Color3.fromRGB(108, 32, 255), -- cor de destaque inicial
	Volume = 0.7,                         -- 0 a 1
	Pitch = 1,                            -- 0.5 a 2
})

-- ABA DE EXPLOITS
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

local ExploitsTab = Window:AddTab("Exploits")

ExploitsTab:AddSection("Controladores de Funções")
-- EXECUÇÃO / DOWNLOAD

ExploitsTab:AddButton("Remote blox", function()
    loadstring(game:HttpGet("https://raw.githubusercontent.com/controlbloxs02/ControlBlox-Raw/refs/heads/main/RAWS/control_blox_remote.luau"))()
	Window:Notify("Control Blox:", "Remote blox!", 3)
end)


Window:SetDesign("DarkPurple")

local Home = Window:AddTab("Início")
Home:AddSection("Geral")
Home:AddLabel("Adicione suas funções nas abas.")
Home:AddButton("Testar notificação", function()
	Window:Notify("Teste", "Funcionando perfeitamente.", 3)
end)



local Player = Window:AddTab("Jogador")
Player:AddSection("Movimento")

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local LocalPlayer = Players.LocalPlayer

-- VELOCIDADE DE CAMINHADA
Player:AddSlider("Velocidade", 16, 100, 16, function(v)
    local character = LocalPlayer.Character
    local hum = character and character:FindFirstChildOfClass("Humanoid")

    if hum then
        hum.WalkSpeed = v
    end
end)

-- FORÇA DO PULO
Player:AddSlider("Pulo", 50, 200, 50, function(v)
    local character = LocalPlayer.Character
    local hum = character and character:FindFirstChildOfClass("Humanoid")

    if hum then
        hum.UseJumpPower = true
        hum.JumpPower = v
    end
end)

-- CONFIGURAÇÃO DO FLY
local flyEnabled = false
local flySpeed = 50

local flyConnection
local bodyVelocity
local bodyGyro
local currentHumanoid
local currentRoot
local previousPlatformStand
local previousAutoRotate
local previousWalkSpeed

local inputStates = {
    W = false,
    A = false,
    S = false,
    D = false,
    Space = false,
    LeftShift = false
}

local function clearInputs()
    for key in pairs(inputStates) do
        inputStates[key] = false
    end
end

local function stopFlying()
    flyEnabled = false

    if flyConnection then
        flyConnection:Disconnect()
        flyConnection = nil
    end

    if bodyVelocity then
        bodyVelocity:Destroy()
        bodyVelocity = nil
    end

    if bodyGyro then
        bodyGyro:Destroy()
        bodyGyro = nil
    end

    if currentHumanoid and currentHumanoid.Parent then
        currentHumanoid.PlatformStand = previousPlatformStand or false
        currentHumanoid.AutoRotate = previousAutoRotate ~= nil
            and previousAutoRotate or true

        if previousWalkSpeed ~= nil then
            currentHumanoid.WalkSpeed = previousWalkSpeed
        end

        currentHumanoid:ChangeState(
            Enum.HumanoidStateType.GettingUp
        )
    end

    currentHumanoid = nil
    currentRoot = nil

    clearInputs()
end

local function startFlying()
    stopFlying()

    local character = LocalPlayer.Character
    local humanoid = character and character:FindFirstChildOfClass("Humanoid")
    local rootPart = character and character:FindFirstChild("HumanoidRootPart")

    if not humanoid or not rootPart then
        return
    end

    currentHumanoid = humanoid
    currentRoot = rootPart

    previousPlatformStand = humanoid.PlatformStand
    previousAutoRotate = humanoid.AutoRotate
    previousWalkSpeed = humanoid.WalkSpeed

    flyEnabled = true

    humanoid.PlatformStand = true
    humanoid.AutoRotate = false

    bodyVelocity = Instance.new("BodyVelocity")
    bodyVelocity.Name = "MinezinFlyVelocity"
    bodyVelocity.MaxForce = Vector3.new(
        math.huge,
        math.huge,
        math.huge
    )
    bodyVelocity.P = 1250
    bodyVelocity.Velocity = Vector3.zero
    bodyVelocity.Parent = rootPart

    bodyGyro = Instance.new("BodyGyro")
    bodyGyro.Name = "MinezinFlyGyro"
    bodyGyro.MaxTorque = Vector3.new(4000, 4000, 4000)
    bodyGyro.P = 3000
    bodyGyro.D = 500
    bodyGyro.CFrame = workspace.CurrentCamera.CFrame
    bodyGyro.Parent = rootPart

    flyConnection = RunService.RenderStepped:Connect(function()
        if not flyEnabled
            or not currentRoot
            or not currentRoot.Parent
            or not currentHumanoid
            or currentHumanoid.Health <= 0 then

            stopFlying()
            return
        end

        local camera = workspace.CurrentCamera
        if not camera then
            bodyVelocity.Velocity = Vector3.zero
            return
        end

        -- A rotação acompanha a câmera
        bodyGyro.CFrame = camera.CFrame

        -- Movimento relativo à câmera
        local direction = Vector3.zero

        if inputStates.W then
            direction += Vector3.new(0, 0, -1)
        end
        if inputStates.S then
            direction += Vector3.new(0, 0, 1)
        end
        if inputStates.A then
            direction += Vector3.new(-1, 0, 0)
        end
        if inputStates.D then
            direction += Vector3.new(1, 0, 0)
        end
        if inputStates.Space then
            direction += Vector3.new(0, 1, 0)
        end
        if inputStates.LeftShift then
            direction += Vector3.new(0, -1, 0)
        end

        if direction.Magnitude > 0 then
            direction = direction.Unit
        end

        local worldDirection =
            camera.CFrame:VectorToWorldSpace(direction)

        bodyVelocity.Velocity = worldDirection * flySpeed
    end)
end

-- TECLAS DO FLY
UserInputService.InputBegan:Connect(function(input, processed)
    if processed or not flyEnabled then
        return
    end

    if input.KeyCode == Enum.KeyCode.W then
        inputStates.W = true
    elseif input.KeyCode == Enum.KeyCode.A then
        inputStates.A = true
    elseif input.KeyCode == Enum.KeyCode.S then
        inputStates.S = true
    elseif input.KeyCode == Enum.KeyCode.D then
        inputStates.D = true
    elseif input.KeyCode == Enum.KeyCode.Space then
        inputStates.Space = true
    elseif input.KeyCode == Enum.KeyCode.LeftShift then
        inputStates.LeftShift = true
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.W then
        inputStates.W = false
    elseif input.KeyCode == Enum.KeyCode.A then
        inputStates.A = false
    elseif input.KeyCode == Enum.KeyCode.S then
        inputStates.S = false
    elseif input.KeyCode == Enum.KeyCode.D then
        inputStates.D = false
    elseif input.KeyCode == Enum.KeyCode.Space then
        inputStates.Space = false
    elseif input.KeyCode == Enum.KeyCode.LeftShift then
        inputStates.LeftShift = false
    end
end)

-- TOGGLE FLY NO SEU MENU
Player:AddToggle("Fly", false, function(state)
    if state then
        startFlying()
    else
        stopFlying()
    end
end)

-- SLIDER DE VELOCIDADE DO VOO: 0 A 100
Player:AddSlider("Velocidade do Fly", 0, 300, 100, function(v)
    flySpeed = v
end)

-- DESLIGA O FLY AO RENASCER
LocalPlayer.CharacterAdded:Connect(function()
    stopFlying()
end)

-- NOCLIP
local noclipEnabled = false
local noclipConnection

local function setNoClip(state)
    noclipEnabled = state

    if noclipConnection then
        noclipConnection:Disconnect()
        noclipConnection = nil
    end

    if state then
        noclipConnection = game:GetService("RunService").Stepped:Connect(function()
            local character = LocalPlayer.Character
            if not character then return end

            for _, part in ipairs(character:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.CanCollide = false
                end
            end
        end)
    else
        local character = LocalPlayer.Character
        if character then
            for _, part in ipairs(character:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.CanCollide = true
                end
            end
        end
    end
end

Player:AddToggle("NoClip", false, function(state)
    setNoClip(state)
end)

-- Desliga ao renascer
LocalPlayer.CharacterAdded:Connect(function()
    if noclipEnabled then
        setNoClip(false)
    end
end)



Player:AddSection("Opções")
Player:AddToggle("Exemplo de toggle", false, function(on)
	Window:Notify("Toggle", on and "Ativado" or "Desativado", 2)
end)
Player:AddKeybind("Atalho", Enum.KeyCode.F, function()
	Window:Notify("Atalho", "Tecla pressionada.", 2)
end)

Window:AddProfileTab("Perfil")

local Config = Window:AddTab("Config")
Config:AddSection("Aparência")
Config:AddColorPicker("Cor de destaque", Color3.fromRGB(108, 32, 255), function(c)
	Window:SetAccent(c)
end)

Config:AddSection("Som")
Config:AddToggle("Sons ativados", true, function(on)
	Window.Settings.Muted = not on
end)
Config:AddSlider("Volume", 0, 100, 70, function(v)
	Window:SetVolume(v / 100)
end)
Config:AddSlider("Tom do som (%)", 50, 200, 100, function(v)
	Window:SetPitch(v / 100)
end)
Config:AddSection("Menu")
Config:AddToggle("Botão flutuante", true, function(on)
	Window:SetButtonVisible(on)
end)


----------------------------------------------------------------
-- ULTRA LOW GRAPHICS - CONTROL BLOX
----------------------------------------------------------------
local Lighting = game:GetService("Lighting")
local Workspace = game:GetService("Workspace")

local graficosAtivos = false
local originais = {}
local objetosModificados = {}

local function guardar(obj, propriedade)
    if not obj or not obj.Parent then
        return
    end

    originais[obj] = originais[obj] or {}

    if originais[obj][propriedade] == nil then
        local ok, valor = pcall(function()
            return obj[propriedade]
        end)

        if ok then
            originais[obj][propriedade] = valor
        end
    end
end

local function alterar(obj, propriedade, valor)
    guardar(obj, propriedade)

    pcall(function()
        obj[propriedade] = valor
    end)
end

local function aplicarEmTudo()
    -- Reduz efeitos de iluminação
    alterar(Lighting, "GlobalShadows", false)
    alterar(Lighting, "FogEnd", 100000)
    alterar(Lighting, "Brightness", 1)

    -- Desativa efeitos visuais
    for _, obj in ipairs(Lighting:GetDescendants()) do
        if obj:IsA("PostEffect") then
            alterar(obj, "Enabled", false)
        elseif obj:IsA("Atmosphere") then
            alterar(obj, "Density", 0)
            alterar(obj, "Haze", 0)
            alterar(obj, "Glare", 0)
        elseif obj:IsA("Clouds") then
            alterar(obj, "Enabled", false)
        end
    end

    -- Deixa peças com materiais simples e sem reflexo
    for _, obj in ipairs(Workspace:GetDescendants()) do
        if obj:IsA("BasePart") then
            alterar(obj, "Material", Enum.Material.SmoothPlastic)
            alterar(obj, "Reflectance", 0)

        elseif obj:IsA("Decal") or obj:IsA("Texture") then
            alterar(obj, "Transparency", 1)

        elseif obj:IsA("ParticleEmitter")
            or obj:IsA("Trail")
            or obj:IsA("Beam") then
            alterar(obj, "Enabled", false)

        elseif obj:IsA("Fire")
            or obj:IsA("Smoke")
            or obj:IsA("Sparkles") then
            alterar(obj, "Enabled", false)

        elseif obj:IsA("PostEffect") then
            alterar(obj, "Enabled", false)
        end
    end
end

local function restaurarTudo()
    for obj, propriedades in pairs(originais) do
        if obj and obj.Parent then
            for propriedade, valor in pairs(propriedades) do
                pcall(function()
                    obj[propriedade] = valor
                end)
            end
        end
    end

    table.clear(originais)
end

local function configurarGraficos(on)
    if graficosAtivos == on then
        return
    end

    graficosAtivos = on

    if on then
        aplicarEmTudo()
        print("[Control Blox] Ultra Low Graphics ativado")
    else
        restaurarTudo()
        print("[Control Blox] Configurações visuais restauradas")
    end
end

----------------------------------------------------------------
-- TOGGLE NA ABA CONFIG
----------------------------------------------------------------
Config:AddToggle("Gráficos no mínimo", false, function(on)
    configurarGraficos(on)
end)

return Library
