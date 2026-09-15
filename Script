--[[
	STEAL AN EGG - Full Predictor + Live Detector
	+ Panel Prediksi Real-Time
	+ Tombol Kirim Ulang Prediksi
	+ Test webhook
	+ Close GUI
]]

local Players     = game:GetService("Players")
local HttpService = game:GetService("HttpService")
local player      = Players.LocalPlayer

--// CONFIG
getgenv().webhookUrl     = ""    -- ⬅️ Paste webhook di sini
getgenv().PredictEgg     = false
getgenv().DetectEgg      = false
getgenv().DetectReset    = false
getgenv().DetectRift     = false
getgenv().AlertRare      = false
getgenv().FilterRarity   = "All"
getgenv().LastPrediction = {}
getgenv().LastSentEgg    = nil   -- untuk kirim ulang

--// GUI
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "SEGCombined"
ScreenGui.Parent = game:GetService("CoreGui")
ScreenGui.ResetOnSpawn = false

local Frame = Instance.new("Frame")
Frame.Parent = ScreenGui
Frame.Size = UDim2.new(0, 320, 0, 660)
Frame.Position = UDim2.new(0, 20, 0, 60)
Frame.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
Frame.BorderSizePixel = 0
Frame.Active = true
Frame.Draggable = true

local Title = Instance.new("TextLabel")
Title.Parent = Frame
Title.Size = UDim2.new(1, 0, 0, 30)
Title.BackgroundColor3 = Color3.fromRGB(35, 35, 45)
Title.Text = "🥚 SEG Predictor + Detector"
Title.TextColor3 = Color3.fromRGB(0, 255, 120)
Title.Font = Enum.Font.SourceSansBold
Title.TextSize = 15
Title.BorderSizePixel = 0

--// Helpers
local function CreateLabel(text, yPos)
	local lbl = Instance.new("TextLabel")
	lbl.Parent = Frame
	lbl.Size = UDim2.new(0, 300, 0, 18)
	lbl.Position = UDim2.new(0, 10, 0, yPos)
	lbl.BackgroundTransparency = 1
	lbl.Text = text
	lbl.TextColor3 = Color3.fromRGB(170, 170, 180)
	lbl.Font = Enum.Font.SourceSansBold
	lbl.TextSize = 12
	lbl.TextXAlignment = Enum.TextXAlignment.Left
	return lbl
end

local function CreateButton(name, yPos, callback)
	local btn = Instance.new("TextButton")
	btn.Parent = Frame
	btn.Size = UDim2.new(0, 300, 0, 26)
	btn.Position = UDim2.new(0, 10, 0, yPos)
	btn.BackgroundColor3 = Color3.fromRGB(50, 50, 60)
	btn.BorderSizePixel = 0
	btn.Text = name
	btn.TextColor3 = Color3.fromRGB(255, 255, 255)
	btn.Font = Enum.Font.SourceSans
	btn.TextSize = 12
	btn.MouseButton1Click:Connect(callback)
	return btn
end

local function CreateInput(placeholder, yPos, callback)
	local box = Instance.new("TextBox")
	box.Parent = Frame
	box.Size = UDim2.new(0, 300, 0, 26)
	box.Position = UDim2.new(0, 10, 0, yPos)
	box.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
	box.BorderSizePixel = 0
	box.Text = ""
	box.PlaceholderText = placeholder
	box.TextColor3 = Color3.fromRGB(255, 255, 255)
	box.PlaceholderColor3 = Color3.fromRGB(130, 130, 140)
	box.Font = Enum.Font.SourceSans
	box.TextSize = 12
	box.ClearTextOnFocus = false
	box.TextXAlignment = Enum.TextXAlignment.Left
	box:GetPropertyChangedSignal("Text"):Connect(function()
		callback(box.Text)
	end)
	return box
end

--// Status
local statusLbl = Instance.new("TextLabel")
statusLbl.Parent = Frame
statusLbl.Size = UDim2.new(1, -20, 0, 20)
statusLbl.Position = UDim2.new(0, 10, 1, -25)
statusLbl.BackgroundTransparency = 1
statusLbl.Text = "Ready."
statusLbl.TextColor3 = Color3.fromRGB(0, 255, 100)
statusLbl.Font = Enum.Font.SourceSans
statusLbl.TextSize = 12
statusLbl.TextXAlignment = Enum.TextXAlignment.Left

local function setStatus(txt, color)
	statusLbl.Text = txt
	statusLbl.TextColor3 = color or Color3.fromRGB(0, 255, 100)
end

--// Webhook
local function sendWebhook(embed, content)
	if not getgenv().webhookUrl or getgenv().webhookUrl == "" then return false end
	local body = HttpService:JSONEncode({
		content = content or "",
		embeds = embed and {embed} or {}
	})
	local req = http_request or request or (syn and syn.request) or (fluxus and fluxus.request) or (http and http.request)
	if not req then return false end
	local ok = pcall(function()
		req({
			Url = getgenv().webhookUrl,
			Method = "POST",
			Headers = {["content-type"] = "application/json"},
			Body = body
		})
	end)
	return ok
end

local function footer()
	return {text = "SEG Predictor | " .. os.date("%Y-%m-%d %H:%M:%S")}
end

--// PREDICTION DATA
local EGG_SCHEDULE = {
	{name = "Kraken Egg",        interval = 1800, rarity = "Rare"},
	{name = "Tyrannosaurus Egg", interval = 2400, rarity = "Rare"},
	{name = "Mosasaurus Egg",    interval = 3000, rarity = "Rare"},
	{name = "Eternal Lunar",     interval = 3600, rarity = "Secret"},
	{name = "Unicorn Egg",       interval = 1200, rarity = "Rare"},
	{name = "Heaven Egg",        interval = 2700, rarity = "Event"},
}

for _, egg in ipairs(EGG_SCHEDULE) do
	if not getgenv().LastPrediction[egg.name] then
		getgenv().LastPrediction[egg.name] = os.time()
	end
end

local function getAllPredictions()
	local now = os.time()
	local list = {}
	for _, egg in ipairs(EGG_SCHEDULE) do
		local lastSeen = getgenv().LastPrediction[egg.name] or now
		local nextTime = lastSeen + egg.interval
		local eta = nextTime - now
		if eta < 0 then eta = 0 end
		table.insert(list, {
			name = egg.name,
			rarity = egg.rarity,
			eta = eta,
			interval = egg.interval
		})
	end
	table.sort(list, function(a, b) return a.eta < b.eta end)
	return list
end

local function fmtTime(sec)
	local m = math.floor(sec / 60)
	local s = sec % 60
	return string.format("%02dm %02ds", m, s)
end

local function getNextPrediction()
	local list = getAllPredictions()
	if #list == 0 then return nil, 0 end
	return list[1], list[1].eta
end

--// FUNGSI KIRIM PREDIKSI (bisa dipanggil berkali-kali)
local function sendPredictionToDiscord()
	local egg, eta = getNextPrediction()
	if not egg then
		setStatus("")
		return false
	end
	-- Simpan untuk resend
	getgenv().LastSentEgg = egg

	-- Bikin list semua egg buat embed
	local allList = getAllPredictions()
	local listText = ""
	for i, e in ipairs(allList) do
		listText = listText .. string.format("%d. **%s** — `%s` (%s)\n", i, e.name, fmtTime(e.eta), e.rarity)
	end

	local ok = sendWebhook({
		title = "🔮 Egg Prediction",
		description = "**Next:** " .. egg.name .. " dalam `" .. fmtTime(eta) .. "`\n\n**Full Prediction List:**\n" .. listText,
		color = 0x00ff64,
		fields = {
			{name = "🥚 Next Egg", value = "`" .. egg.name .. "`", inline = true},
			{name = "🎯 Rarity", value = "`" .. egg.rarity .. "`", inline = true},
			{name = "⏰ ETA", value = "`" .. fmtTime(eta) .. "`", inline = true}
		},
		footer = footer()
	})
	return ok
end

--// ============ LIVE PREDICTION PANEL ============
local PanelFrame = Instance.new("Frame")
PanelFrame.Parent = Frame
PanelFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 20)
PanelFrame.BorderSizePixel = 0
PanelFrame.Position = UDim2.new(0, 10, 0, 200)
PanelFrame.Size = UDim2.new(0, 300, 0, 175)

local PanelTitle = Instance.new("TextLabel")
PanelTitle.Parent = PanelFrame
PanelTitle.Size = UDim2.new(1, 0, 0, 20)
PanelTitle.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
PanelTitle.Text = "📊 PREDIKSI LIVE"
PanelTitle.TextColor3 = Color3.fromRGB(0, 255, 120)
PanelTitle.Font = Enum.Font.SourceSansBold
PanelTitle.TextSize = 12
PanelTitle.BorderSizePixel = 0

local rowContainer = Instance.new("Frame")
rowContainer.Parent = PanelFrame
rowContainer.Position = UDim2.new(0, 5, 0, 22)
rowContainer.Size = UDim2.new(1, -10, 1, -25)
rowContainer.BackgroundTransparency = 1

local rows = {}
local function makeRow(index, yPos)
	local row = Instance.new("Frame")
	row.Parent = rowContainer
	row.Position = UDim2.new(0, 0, 0, yPos)
	row.Size = UDim2.new(1, 0, 0, 23)
	row.BackgroundColor3 = Color3.fromRGB(25, 25, 32)
	row.BorderSizePixel = 0

	local nameLbl = Instance.new("TextLabel")
	nameLbl.Parent = row
	nameLbl.Position = UDim2.new(0, 5, 0, 0)
	nameLbl.Size = UDim2.new(0.55, 0, 1, 0)
	nameLbl.BackgroundTransparency = 1
	nameLbl.Text = "Egg Name"
	nameLbl.TextColor3 = Color3.fromRGB(230, 230, 230)
	nameLbl.Font = Enum.Font.SourceSans
	nameLbl.TextSize = 12
	nameLbl.TextXAlignment = Enum.TextXAlignment.Left

	local etaLbl = Instance.new("TextLabel")
	etaLbl.Parent = row
	etaLbl.Position = UDim2.new(0.55, 0, 0, 0)
	etaLbl.Size = UDim2.new(0.30, 0, 1, 0)
	etaLbl.BackgroundTransparency = 1
	etaLbl.Text = "--:--"
	etaLbl.TextColor3 = Color3.fromRGB(255, 200, 0)
	etaLbl.Font = Enum.Font.SourceSansBold
	etaLbl.TextSize = 12
	etaLbl.TextXAlignment = Enum.TextXAlignment.Center

	local rarLbl = Instance.new("TextLabel")
	rarLbl.Parent = row
	rarLbl.Position = UDim2.new(0.85, 0, 0, 0)
	rarLbl.Size = UDim2.new(0.15, 0, 1, 0)
	rarLbl.BackgroundTransparency = 1
	rarLbl.Text = "?"
	rarLbl.TextColor3 = Color3.fromRGB(150, 150, 150)
	rarLbl.Font = Enum.Font.SourceSans
	rarLbl.TextSize = 11
	rarLbl.TextXAlignment = Enum.TextXAlignment.Right

	return {frame = row, name = nameLbl, eta = etaLbl, rarity = rarLbl}
end

for i = 1, #EGG_SCHEDULE do
	rows[i] = makeRow(i, (i - 1) * 25)
end

local function rarityColor(r)
	if r == "Secret" then return Color3.fromRGB(255, 80, 200) end
	if r == "Eternal" then return Color3.fromRGB(255, 180, 0) end
	if r == "Divine" then return Color3.fromRGB(180, 80, 255) end
	if r == "Event" then return Color3.fromRGB(80, 180, 255) end
	if r == "Rare" then return Color3.fromRGB(100, 255, 100) end
	return Color3.fromRGB(180, 180, 180)
end

local function updatePanel()
	local list = getAllPredictions()
	for i, row in ipairs(rows) do
		if list[i] then
			row.name.Text = list[i].name
			row.eta.Text = fmtTime(list[i].eta)
			row.rarity.Text = list[i].rarity
			row.rarity.TextColor3 = rarityColor(list[i].rarity)

			if i == 1 then
				row.frame.BackgroundColor3 = Color3.fromRGB(0, 60, 30)
				row.eta.TextColor3 = Color3.fromRGB(0, 255, 120)
			else
				row.frame.BackgroundColor3 = Color3.fromRGB(25, 25, 32)
				row.eta.TextColor3 = Color3.fromRGB(255, 200, 0)
			end
		else
			row.name.Text = "-"
			row.eta.Text = "--:--"
			row.rarity.Text = ""
		end
	end
end

--// ============ PARSER notifikasi ============
local function parseNotification(text)
	local eggName, biome = text:match("A (.+) Egg spawned in (.+)!")
	if eggName then
		local rarity = "Unknown"
		if text:find("Secret") then rarity = "Secret"
		elseif text:find("Eternal") then rarity = "Eternal"
		elseif text:find("Divine") then rarity = "Divine"
		elseif text:find("Rare") then rarity = "Rare"
		end
		return {type = "egg", egg = eggName, biome = biome, rarity = rarity, raw = text}
	end
	if text:find("ALL EGG RESET") then return {type = "reset", raw = text} end
	if text:find("Rift") and (text:find("spawn") or text:find("open") or text:find("event")) then
		return {type = "rift", raw = text}
	end
	return nil
end

--// ============ HANDLER notifikasi ============
local function handleNotification(notif)
	if not notif then return end

	if notif.type == "egg" then
		if not getgenv().DetectEgg then return end
		if getgenv().FilterRarity ~= "All" and notif.rarity ~= getgenv().FilterRarity then return end

		getgenv().LastPrediction[notif.egg] = os.time()
		updatePanel()

		setStatus("🥚 " .. notif.egg .. " @ " .. notif.biome, Color3.fromRGB(255, 200, 0))
		sendWebhook({
			title = "🥚 EGG SPAWNED!",
			description = "**" .. notif.egg .. "** muncul di **" .. notif.biome .. "**",
			color = notif.rarity == "Divine" and 0xff00ff or (notif.rarity == "Eternal" and 0xffaa00 or 0x00ff64),
			fields = {
				{name = "Egg", value = "`" .. notif.egg .. "`", inline = true},
				{name = "Biome", value = "`" .. notif.biome .. "`", inline = true},
				{name = "Rarity", value = "`" .. notif.rarity .. "`", inline = true}
			},
			footer = footer()
		})
	elseif notif.type == "reset" then
		if not getgenv().DetectReset then return end
		for _, egg in ipairs(EGG_SCHEDULE) do
			getgenv().LastPrediction[egg.name] = os.time()
		end
		updatePanel()
		setStatus("🔄 ALL EGG RESET!", Color3.fromRGB(255, 100, 100))
		sendWebhook({
			title = "🔄 ALL EGG RESET!",
			description = "Semua egg di-reset!",
			color = 0xff6b6b,
			footer = footer()
		})
	elseif notif.type == "rift" then
		if not getgenv().DetectRift then return end
		setStatus("🌀 RIFT EVENT!", Color3.fromRGB(150, 100, 255))
		sendWebhook({
			title = "🌀 RIFT EVENT!",
			description = "Rift event terbuka!",
			color = 0x9b59b6,
			footer = footer()
		})
	end
end

--// HOOK SYSTEM CHAT
local function hookSystemChat()
	local TC = game:GetService("TextChatService")
	if TC.ChatVersion == Enum.ChatVersion.TextChatService then
		TC.MessageReceived:Connect(function(msg)
			local text = msg.Text or ""
			local notif = parseNotification(text)
			if notif then
				local source = msg.TextSource
				if not source or source.Name == "System" or source.Name == "Roblox" then
					handleNotification(notif)
				end
			end
		end)
	end

	local RP = game:GetService("ReplicatedStorage")
	local defaultChat = RP:FindFirstChild("DefaultChatSystemChatEvents")
	if defaultChat then
		local onMessage = defaultChat:FindFirstChild("OnMessageDoneFiltering")
		if onMessage then
			onMessage.OnClientEvent:Connect(function(data)
				local text = data.Message or ""
				local notif = parseNotification(text)
				if notif and (data.FromSpeaker == "System" or data.FromSpeaker == "Roblox") then
					handleNotification(notif)
				end
			end)
		end
	end
end

pcall(hookSystemChat)

--// ============ UI ============
CreateLabel("Discord Webhook:", 38)
CreateInput("https://discord.com/api/webhooks/...", 58, function(txt)
	getgenv().webhookUrl = txt
end)

CreateLabel("Test Webhook:", 92)
local y = 112

CreateButton("🧪 Test Ping (cek koneksi)", y, function()
	if getgenv().webhookUrl == "" then
		setStatus("❌ Webhook URL kosong!", Color3.fromRGB(255, 80, 80))
		return
	end
	setStatus("Mengirim test ping...", Color3.fromRGB(255, 200, 0))
	local ok = sendWebhook({
		title = "🧪 Test Ping",
		description = "Webhook berhasil terhubung!",
		color = 0x2ecc71,
		footer = footer()
	})
	if ok then setStatus("✅ Test ping terkirim!", Color3.fromRGB(0, 255, 100)) end
end)
y = y + 30

CreateButton("📨 Test Egg Spawn Notif", y, function()
	if getgenv().webhookUrl == "" then
		setStatus("❌ Webhook URL kosong!", Color3.fromRGB(255, 80, 80))
		return
	end
	sendWebhook({
		title = "🥚 EGG SPAWNED! (Test)",
		description = "**Test Dragon Egg** muncul di **Cosmic**",
		color = 0xffaa00,
		fields = {
			{name = "Egg", value = "`Test Dragon Egg`", inline = true},
			{name = "Biome", value = "`Cosmic`", inline = true},
			{name = "Rarity", value = "`Secret`", inline = true}
		},
		footer = footer()
	})
	setStatus("✅ Test egg notif terkirim!", Color3.fromRGB(0, 255, 100))
end)
y = y + 34

-- ===== KONTROL PREDIKSI =====
CreateLabel("Kontrol Prediksi:", 390)

y = 410
local PredictBtn = CreateButton("🔮 Predict Live: OFF", y, function()
	getgenv().PredictEgg = not getgenv().PredictEgg
	PredictBtn.Text = "🔮 Predict Live: " .. (getgenv().PredictEgg and "ON" or "OFF")
	PredictBtn.BackgroundColor3 = getgenv().PredictEgg and Color3.fromRGB(0, 130, 0) or Color3.fromRGB(50, 50, 60)
	if getgenv().PredictEgg then updatePanel() end
end)
y = y + 30

-- TOMBOL KIRIM PERTAMA KALI
CreateButton("📤 Kirim Prediksi ke Discord", y, function()
	if getgenv().webhookUrl == "" then
		setStatus("❌ Webhook URL kosong!", Color3.fromRGB(255, 80, 80))
		return
	end
	setStatus("Mengirim prediksi...", Color3.fromRGB(255, 200, 0))
	local ok = sendPredictionToDiscord()
	if ok then
		setStatus("✅ Prediksi terkirim!", Color3.fromRGB(0, 255, 100))
	else
		setStatus("❌ Gagal kirim prediksi", Color3.fromRGB(255, 80, 80))
	end
end)
y = y + 30

-- ⬇️ TOMBOL BARU: KIRIM ULANG ⬇️
CreateButton("🔄 Kirim Ulang Prediksi (Resend)", y, function()
	if getgenv().webhookUrl == "" then
		setStatus("❌ Webhook URL kosong!", Color3.fromRGB(255, 80, 80))
		return
	end
	if not getgenv().LastSentEgg then
		setStatus("⚠️ Belum ada prediksi yang dikirim sebelumnya", Color3.fromRGB(255, 200, 0))
		return
	end
	setStatus("Mengirim ulang prediksi...", Color3.fromRGB(255, 200, 0))
	local ok = sendPredictionToDiscord()
	if ok then
		setStatus("🔄 Prediksi terkirim ULANG!", Color3.fromRGB(0, 255, 100))
	else
		setStatus("❌ Gagal kirim ulang", Color3.fromRGB(255, 80, 80))
	end
end)
y = y + 34

-- ===== LIVE DETECTOR =====
CreateLabel("Live Detector:", y)
y = y + 22

local EggDetBtn = CreateButton("🥚 Detect Egg Spawn: OFF", y, function()
	getgenv().DetectEgg = not getgenv().DetectEgg
	EggDetBtn.Text = "🥚 Detect Egg Spawn: " .. (getgenv().DetectEgg and "ON" or "OFF")
	EggDetBtn.BackgroundColor3 = getgenv().DetectEgg and Color3.fromRGB(0, 130, 0) or Color3.fromRGB(50, 50, 60)
end)
y = y + 30

local ResetDetBtn = CreateButton("🔄 Detect Egg Reset: OFF", y, function()
	getgenv().DetectReset = not getgenv().DetectReset
	ResetDetBtn.Text = "🔄 Detect Egg Reset: " .. (getgenv().DetectReset and "ON" or "OFF")
	ResetDetBtn.BackgroundColor3 = getgenv().DetectReset and Color3.fromRGB(0, 130, 0) or Color3.fromRGB(50, 50, 60)
end)
y = y + 30

local RiftDetBtn = CreateButton("🌀 Detect Rift Event: OFF", y, function()
	getgenv().DetectRift = not getgenv().DetectRift
	RiftDetBtn.Text = "🌀 Detect Rift Event: " .. (getgenv().DetectRift and "ON" or "OFF")
	RiftDetBtn.BackgroundColor3 = getgenv().DetectRift and Color3.fromRGB(0, 130, 0) or Color3.fromRGB(50, 50, 60)
end)
y = y + 34

local RarityCycle = {"All", "Secret", "Eternal", "Divine"}
local rarityIndex = 1
local RarityBtn = CreateButton("🎯 Rarity Filter: All", y, function()
	rarityIndex = rarityIndex + 1
	if rarityIndex > #RarityCycle then rarityIndex = 1 end
	getgenv().FilterRarity = RarityCycle[rarityIndex]
	RarityBtn.Text = "🎯 Rarity Filter: " .. getgenv().FilterRarity
end)
y = y + 34

CreateButton("❌ Close GUI", y, function()
	ScreenGui:Destroy()
	print("✅ SEG GUI closed")
end)
y = y + 30

Frame.Size = UDim2.new(0, 320, 0, y + 55)

--// AUTO LOOP
task.spawn(function()
	while task.wait(1) do
		if not ScreenGui.Parent then break end

		if getgenv().PredictEgg then
			updatePanel()

			local egg, eta = getNextPrediction()
			if egg and getgenv().AlertRare and egg.rarity == "Secret" and eta <= 60 and eta > 0 then
				sendWebhook({
					title = "🚨 SECRET EGG INCOMING!",
					description = "**" .. egg.name .. "** dalam **" .. fmtTime(eta) .. "**!",
					color = 0xff0000,
					footer = footer()
				})
				setStatus("🚨 SECRET EGG ALERT TERKIRIM!", Color3.fromRGB(255, 50, 50))
			end
		end
	end
end)

updatePanel()

print("✅ SEG Predictor + Live Panel Loaded")
