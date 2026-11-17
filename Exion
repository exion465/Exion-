if not LPH_OBFUSCATED then
	local function Pass(...)
		return ...
	end
	local function Nothing() end
	LPH_ENCSTR = Pass
	LPH_STRENC = Pass
	LPH_ENCNUM = Pass
	LPH_NUMENC = Pass
	LPH_CRASH = Nothing
	LPH_PRECHECK = function(check)
		check()
	end
	LPH_REWRITE = Pass
	LPH_ATTRIBUTES = Nothing
	ENCRYPT = Nothing
	VM = Nothing
	PRESET = Nothing
	OPTIMIZE = Nothing
	NO_UPVALUES = Nothing
	ERROR_HANDLING = Nothing
	UNROLL = Nothing
	INLINE = Nothing
	TRANSFORM = Nothing
	NONE = Nothing
	OPAL = Nothing
	ONYX = Nothing
	FAST = Nothing
	BALANCED = Nothing
	SECURE = Nothing
	EXTRACT = Nothing
	CONTROL_FLOW = Nothing
	REWRITE_NAMECALLS = Nothing
	GLOBALS = Nothing
	CONSTANTS = Nothing
end

local TweenService = game:GetService("TweenService")
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TextService = game:GetService("TextService")
local Lighting = game:GetService("Lighting")
local HttpService = game:GetService("HttpService")
local GuiService = game:GetService("GuiService")

local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

local IsMobileDevice = UserInputService.TouchEnabled and not UserInputService.MouseEnabled

local NullUI = {}
NullUI.__index = NullUI
NullUI.Version = "4.0.0"
NullUI.Flags = {}
NullUI._Windows = {}

local function GetGlobalTable()
	local ok, g = pcall(function()
		if getgenv then
			return getgenv()
		end
		return _G
	end)
	return (ok and g) or _G
end

do
	local globalTable = GetGlobalTable()
	local previousUnload = globalTable.__NullUI_Unload
	globalTable.__NullUI_Unload = nil
	if type(previousUnload) == "function" then
		pcall(previousUnload)
	end
end

local Janitor = {}
Janitor.__index = Janitor

function Janitor.new()
	return setmetatable({ _items = {}, _dead = false }, Janitor)
end

function Janitor:Add(item)
	if self._dead then
		if typeof(item) == "RBXScriptConnection" then
			item:Disconnect()
		elseif typeof(item) == "Instance" then
			item:Destroy()
		end
		return item
	end
	table.insert(self._items, item)
	return item
end

function Janitor:Destroy()
	if self._dead then
		return
	end
	self._dead = true
	for i = #self._items, 1, -1 do
		local item = self._items[i]
		self._items[i] = nil
		local t = typeof(item)
		if t == "RBXScriptConnection" then
			pcall(function()
				item:Disconnect()
			end)
		elseif t == "Instance" then
			pcall(function()
				item:Destroy()
			end)
		elseif t == "function" then
			pcall(item)
		elseif t == "table" and type(item.Destroy) == "function" then
			pcall(function()
				item:Destroy()
			end)
		elseif t == "table" and type(item.Disconnect) == "function" then
			pcall(function()
				item:Disconnect()
			end)
		end
	end
end

local IdentitySetter = (function()
	local ok, fn = pcall(function()
		local env = (getgenv and getgenv()) or _G or {}
		return env.setthreadidentity or env.set_thread_identity or env.setidentity
	end)
	if ok and type(fn) == "function" then
		return fn
	end
	return nil
end)()

local function RestoreIdentity()
	if IdentitySetter then
		pcall(IdentitySetter, 8)
	end
end

local function SafeSpawn(fn, ...)
	return task.spawn(function(...)
		RestoreIdentity()
		return fn(...)
	end, ...)
end

local function SafeDefer(fn, ...)
	return task.defer(function(...)
		RestoreIdentity()
		return fn(...)
	end, ...)
end

local function SafeDelay(seconds, fn, ...)
	return task.delay(seconds, function(...)
		RestoreIdentity()
		return fn(...)
	end, ...)
end

local LibJanitor = Janitor.new()

local function MakeSignal()
	local listeners = {}
	return {
		Fire = function(...)
			for _, fn in ipairs(table.clone(listeners)) do
				SafeSpawn(fn, ...)
			end
		end,
		Connect = function(fn)
			table.insert(listeners, fn)
			return {
				Disconnect = function()
					local i = table.find(listeners, fn)
					if i then
						table.remove(listeners, i)
					end
				end,
			}
		end,
		Clear = function()
			table.clear(listeners)
		end,
	}
end

local function SafeClamp(value, lo, hi)
	LPH_ATTRIBUTES(VM(NONE))
	if hi < lo then
		return lo
	end
	return math.clamp(value, lo, hi)
end

local function SafeAlpha(value, min, max)
	LPH_ATTRIBUTES(VM(NONE))
	local range = max - min
	if range == 0 then
		return 0
	end
	return math.clamp((value - min) / range, 0, 1)
end

local function SnapToIncrement(raw, min, max, increment)
	LPH_ATTRIBUTES(VM(NONE))
	if increment <= 0 then
		increment = 1
	end
	local snapped = math.floor((raw - min) / increment + 0.5) * increment + min
	snapped = math.clamp(snapped, min, max)
	local decimals = 0
	local probe = increment
	while decimals < 6 and math.abs(probe - math.floor(probe + 0.5)) > 1e-9 do
		probe = probe * 10
		decimals = decimals + 1
	end
	local factor = 10 ^ decimals
	return math.floor(snapped * factor + (snapped >= 0 and 0.5 or -0.5)) / factor
end

local function FormatNumber(v)
	LPH_ATTRIBUTES(VM(NONE))
	if math.abs(v - math.floor(v + 0.5)) < 1e-9 then
		return tostring(math.floor(v + 0.5))
	end
	return string.format("%.4g", v)
end

local ACRYLIC_DOF_NAME = "NullUI_AcrylicDOF"
local ACRYLIC_DISTANCE = 0.001
local ACRYLIC_TRANSPARENCY = 0.98
local AcrylicDOF = nil
local AcrylicControllers = {}
local CameraConnections = {}
local AcrylicShuttingDown = false

NullUI.Config = { Blur = true, MaxNotifications = 5, LiquidGlass = false }

local function DisconnectCameraSignals()
	for i = #CameraConnections, 1, -1 do
		CameraConnections[i]:Disconnect()
		CameraConnections[i] = nil
	end
end

local function EnsureAcrylicDOF()
	if AcrylicDOF and AcrylicDOF.Parent then
		return AcrylicDOF
	end
	local stale = Lighting:FindFirstChild(ACRYLIC_DOF_NAME)
	if stale then
		stale:Destroy()
	end

	local dof = Instance.new("DepthOfFieldEffect")
	dof.Name = ACRYLIC_DOF_NAME
	dof.FarIntensity = 0
	dof.FocusDistance = 0.05
	dof.InFocusRadius = 0.1
	dof.NearIntensity = 1
	dof.Enabled = false
	dof.Parent = Lighting
	AcrylicDOF = dof
	LibJanitor:Add(dof)
	return dof
end

local function RefreshAcrylicEffect()
	if AcrylicShuttingDown then
		return
	end
	local dof = EnsureAcrylicDOF()
	local active = false
	if NullUI.Config.Blur ~= false then
		for index = 1, #AcrylicControllers do
			if AcrylicControllers[index]._vis then
				active = true
				break
			end
		end
	end
	if dof.Enabled ~= active then
		dof.Enabled = active
	end
end

local function HasVisibleAcrylic()
	for index = 1, #AcrylicControllers do
		if AcrylicControllers[index]._vis then
			return true
		end
	end
	return false
end

local function UpdateAllAcrylic()
	for index = #AcrylicControllers, 1, -1 do
		local controller = AcrylicControllers[index]
		if controller.Destroyed then
			table.remove(AcrylicControllers, index)
		else
			controller:Update()
		end
	end
end

local ACRYLIC_RENDER_KEY = "NullUI_AcrylicFollow"
local AcrylicBound = false

local function UnbindAcrylicRender()
	if not AcrylicBound then
		return
	end
	AcrylicBound = false
	pcall(function()
		RunService:UnbindFromRenderStep(ACRYLIC_RENDER_KEY)
	end)
end

local function BindAcrylicRender()
	if AcrylicBound or AcrylicShuttingDown then
		return
	end
	AcrylicBound = true
	-- Runs right after the camera is updated so the glass panel lands on the same
	-- frame as the window instead of trailing one frame behind it.
	RunService:BindToRenderStep(ACRYLIC_RENDER_KEY, Enum.RenderPriority.Camera.Value + 1, function()
		if NullUI.Config.Blur == false or not HasVisibleAcrylic() then
			return
		end
		UpdateAllAcrylic()
	end)
end

local function BindAcrylicCamera(camera)
	DisconnectCameraSignals()
	if not camera then
		UnbindAcrylicRender()
		return
	end

	table.insert(CameraConnections, camera:GetPropertyChangedSignal("ViewportSize"):Connect(UpdateAllAcrylic))
	BindAcrylicRender()
	UpdateAllAcrylic()
end

local function CreateWindowAcrylic(guiObject)
	local folder = Instance.new("Folder")
	folder.Name = "NullUI_AcrylicWindow"

	local part = Instance.new("Part")
	part.Name = "Glass"
	part.Anchored = true
	part.CanCollide = false
	part.CanQuery = false
	part.CanTouch = false
	part.CastShadow = false
	part.Locked = true
	part.Material = Enum.Material.Glass
	part.Color = Color3.new(0, 0, 0)
	part.Reflectance = 0
	part.Size = Vector3.new(1, 1, 0.001)
	part.Transparency = 1
	part.Parent = folder

	local mesh = Instance.new("SpecialMesh")
	mesh.Name = "AcrylicMesh"
	mesh.MeshType = Enum.MeshType.Brick
	mesh.Offset = Vector3.new(0, 0, -0.000001)
	mesh.Scale = Vector3.new(1, 1, 0.001)
	mesh.Parent = part

	local controller = {
		Gui = guiObject,
		Folder = folder,
		Part = part,
		Mesh = mesh,
		Connections = {},
		Destroyed = false,
	}

	function controller:Update()
		if self.Destroyed then
			return
		end
		local camera = workspace.CurrentCamera
		local gui = self.Gui
		if not camera or not gui or not gui.Parent then
			if self._vis ~= false then
				self._vis = false
				self.Part.Transparency = 1
				RefreshAcrylicEffect()
			end
			return
		end

		if self.Folder.Parent ~= camera then
			self.Folder.Parent = camera
		end

		local size = gui.AbsoluteSize
		local visible = NullUI.Config.Blur ~= false and gui.Visible and size.X > 2 and size.Y > 2
		visible = visible and true or false
		if self._vis ~= visible then
			self._vis = visible
			self.Part.Transparency = visible and ACRYLIC_TRANSPARENCY or 1
			RefreshAcrylicEffect()
		end
		if not visible then
			return
		end

		-- Nothing moved since the last frame: skip the projection maths entirely.
		local camCF = camera:GetRenderCFrame()
		local pos = gui.AbsolutePosition
		local fov = camera.FieldOfView
		if self._cCF == camCF and self._cFov == fov and self._cPos == pos and self._cSize == size then
			return
		end
		self._cCF, self._cFov, self._cPos, self._cSize = camCF, fov, pos, size

		local edgeInset = math.clamp(camera.ViewportSize.Y * 0.012, 8, 18)
		local position = gui.AbsolutePosition + Vector2.new(edgeInset, edgeInset)
		local panelSize = Vector2.new(math.max(1, size.X - edgeInset * 2), math.max(1, size.Y - edgeInset * 2))

		local function ScreenToWorld(point)
			local ray = camera:ScreenPointToRay(point.X, point.Y)
			return ray.Origin + ray.Direction * ACRYLIC_DISTANCE
		end

		local topLeft3D = ScreenToWorld(position)
		local topRight3D = ScreenToWorld(position + Vector2.new(panelSize.X, 0))
		local bottomRight3D = ScreenToWorld(position + panelSize)
		local width = (topRight3D - topLeft3D).Magnitude
		local height = (bottomRight3D - topRight3D).Magnitude
		local renderCFrame = camCF

		self.Part.CFrame = CFrame.fromMatrix(
			(topLeft3D + bottomRight3D) / 2,
			renderCFrame.XVector,
			renderCFrame.YVector,
			renderCFrame.ZVector
		)
		self.Mesh.Scale = Vector3.new(width, height, 0.001)
	end

	function controller:Destroy()
		if self.Destroyed then
			return
		end
		self.Destroyed = true
		for _, connection in ipairs(self.Connections) do
			connection:Disconnect()
		end
		table.clear(self.Connections)
		local index = table.find(AcrylicControllers, self)
		if index then
			table.remove(AcrylicControllers, index)
		end
		if self.Folder then
			self.Folder:Destroy()
		end
		RefreshAcrylicEffect()
	end

	for _, property in ipairs({ "AbsolutePosition", "AbsoluteSize", "Visible" }) do
		table.insert(
			controller.Connections,
			guiObject:GetPropertyChangedSignal(property):Connect(function()
				controller:Update()
			end)
		)
	end
	table.insert(
		controller.Connections,
		guiObject.AncestryChanged:Connect(function()
			if not guiObject.Parent then
				controller:Destroy()
			end
		end)
	)

	table.insert(AcrylicControllers, controller)
	RefreshAcrylicEffect()
	controller:Update()
	return controller
end

BindAcrylicCamera(workspace.CurrentCamera)
LibJanitor:Add(workspace:GetPropertyChangedSignal("CurrentCamera"):Connect(function()
	BindAcrylicCamera(workspace.CurrentCamera)
end))
LibJanitor:Add(function()
	DisconnectCameraSignals()
end)

function NullUI:SetBlurEnabled(enabled)
	NullUI.Config.Blur = enabled and true or false
	RefreshAcrylicEffect()
	UpdateAllAcrylic()
end

local function StopAcrylicRender()
	UnbindAcrylicRender()
	DisconnectCameraSignals()
end

local function DestroyAllAcrylicControllers()
	StopAcrylicRender()
	for index = #AcrylicControllers, 1, -1 do
		AcrylicControllers[index]:Destroy()
	end
	table.clear(AcrylicControllers)
end

local ASSETS_FOLDER = "NullUI/Assets"

local function hasFn(name)
	local ok, fn = pcall(function()
		if getgenv then
			local v = getgenv()[name]
			if type(v) == "function" then
				return v
			end
		end
		if getfenv then
			local v = getfenv(1)[name]
			if type(v) == "function" then
				return v
			end
		end
		return _G[name]
	end)
	if ok and type(fn) == "function" then
		return fn
	end
	return nil
end

local fn_isfolder = hasFn("isfolder")
local fn_makefolder = hasFn("makefolder")
local fn_isfile = hasFn("isfile")
local fn_writefile = hasFn("writefile")
local fn_readfile = hasFn("readfile")
local fn_delfile = hasFn("delfile")
local fn_listfiles = hasFn("listfiles")
local fn_customasset = hasFn("getcustomasset") or hasFn("getsynasset")

local function EnsureAssetsFolder()
	if not (fn_isfolder and fn_makefolder) then
		return false
	end
	local ok = pcall(function()
		if not fn_isfolder("NullUI") then
			fn_makefolder("NullUI")
		end
		if not fn_isfolder(ASSETS_FOLDER) then
			fn_makefolder(ASSETS_FOLDER)
		end
	end)
	return ok
end

local RunCountPath = "NullUI/RunCount.txt"

local function BumpRunCount()
	local count = 1
	if fn_isfile and fn_readfile and fn_isfile(RunCountPath) then
		local ok, data = pcall(fn_readfile, RunCountPath)
		local n = ok and tonumber(data)
		if n then
			count = math.floor(n) + 1
		end
	end
	if fn_writefile then
		EnsureAssetsFolder()
		pcall(fn_writefile, RunCountPath, tostring(count))
	end
	return count
end

local function GetExecutorName()
	local ok, name, version = pcall(function()
		if identifyexecutor then
			return identifyexecutor()
		end
		if getexecutorname then
			return getexecutorname()
		end
		if syn and syn.get_executor_name then
			return syn.get_executor_name()
		end
		return nil
	end)
	if ok and name and name ~= "" then
		return version and version ~= "" and (tostring(name) .. " " .. tostring(version)) or tostring(name)
	end
	return "Unknown"
end

local function FormatClock(minutesAfterMidnight)
	minutesAfterMidnight = minutesAfterMidnight or 0
	local h = math.floor(minutesAfterMidnight / 60) % 24
	local m = math.floor(minutesAfterMidnight % 60)
	local suffix = h >= 12 and "PM" or "AM"
	local h12 = h % 12
	if h12 == 0 then
		h12 = 12
	end
	return string.format("%02d:%02d %s", h12, m, suffix)
end

local function LooksLikeFontFile(data)
	if type(data) ~= "string" or #data < 4096 then
		return false
	end
	local sig = data:sub(1, 4)
	return sig == "\0\1\0\0" or sig == "OTTO" or sig == "true" or sig == "ttcf" or sig == "wOFF" or sig == "wOF2"
end

local function DownloadFontFile(path, url)
	if not (fn_isfile and fn_writefile) then
		return false
	end

	local cached = nil
	if fn_isfile(path) and fn_readfile then
		local ok, data = pcall(fn_readfile, path)
		if ok and LooksLikeFontFile(data) then
			return true
		end
		cached = ok and data or nil
	end

	if cached ~= nil and fn_delfile then
		pcall(fn_delfile, path)
	end

	local ok, body = pcall(function()
		return game:HttpGet(url)
	end)
	if not ok or not LooksLikeFontFile(body) then
		return false
	end

	local wrote = pcall(fn_writefile, path, body)
	return wrote
end

local function LoadCustomFigtree()
	if not (fn_customasset and fn_writefile) then
		return nil
	end

	local okFolder = EnsureAssetsFolder()
	if not okFolder then
		return nil
	end

	local base = LPH_ENCSTR("https://raw.githubusercontent.com/Skinny-yz/NullUI-Assets/main/fonts/")
	local semiPath = ASSETS_FOLDER .. "/Figtree-SemiBold.ttf"
	local regPath = ASSETS_FOLDER .. "/Figtree-Medium.ttf"

	if not DownloadFontFile(semiPath, base .. "Figtree-SemiBold.ttf") then
		return nil
	end
	local hasRegular = DownloadFontFile(regPath, base .. "Figtree-Medium.ttf")
	if not hasRegular then
		regPath = semiPath
	end

	local result = nil
	pcall(function()
		local family = {
			name = "Figtree",
			faces = {
				{ name = "Regular", weight = 400, style = "normal", assetId = fn_customasset(regPath) },
				{ name = "SemiBold", weight = 600, style = "normal", assetId = fn_customasset(semiPath) },
			},
		}
		local familyPath = ASSETS_FOLDER .. "/Figtree.font"
		fn_writefile(familyPath, HttpService:JSONEncode(family))

		local asset = fn_customasset(familyPath)
		result = {
			Regular = Font.new(asset, Enum.FontWeight.Regular, Enum.FontStyle.Normal),
			SemiBold = Font.new(asset, Enum.FontWeight.SemiBold, Enum.FontStyle.Normal),
		}
	end)

	return result
end

local function LoadFonts()

	local custom = LoadCustomFigtree()
	if custom and custom.Regular and custom.SemiBold then
		return custom
	end

	local ok, native = pcall(function()
		return {
			Regular = Font.new(
				"rbxasset://fonts/families/Figtree.json",
				Enum.FontWeight.Regular,
				Enum.FontStyle.Normal
			),
			SemiBold = Font.new(
				"rbxasset://fonts/families/Figtree.json",
				Enum.FontWeight.SemiBold,
				Enum.FontStyle.Normal
			),
		}
	end)
	if ok and native then
		return native
	end

	return {
		Regular = Font.fromEnum(Enum.Font.Gotham),
		SemiBold = Font.fromEnum(Enum.Font.GothamSemibold),
	}
end

local Fonts = LoadFonts()

NullUI.Theme = {
	Background = Color3.fromRGB(16, 16, 16),
	Surface = Color3.fromRGB(24, 24, 24),
	Text = Color3.fromRGB(240, 240, 240),
	TextDim = Color3.fromRGB(150, 150, 155),
	Accent = Color3.fromRGB(255, 255, 255),
	Danger = Color3.fromRGB(205, 205, 210),

	Font = Fonts.SemiBold,
	FontRegular = Fonts.Regular,

	MeasureFont = Enum.Font.GothamSemibold,

	CornerRadius = 16,
	CornerRadiusSm = 8,
	Margin = 14,
	AnimFast = 0.15,
	AnimSlow = 0.32,
}

-- Base surfaces stay neutral; only the accent carries the colour, the way Fluent
-- ships one dark shell with a long list of accent choices.
local THEME_BASES = {
	Dark = {
		Background = Color3.fromRGB(16, 16, 16),
		Surface = Color3.fromRGB(24, 24, 24),
		Text = Color3.fromRGB(240, 240, 240),
		TextDim = Color3.fromRGB(150, 150, 155),
	},
	Darker = {
		Background = Color3.fromRGB(8, 8, 9),
		Surface = Color3.fromRGB(15, 15, 17),
		Text = Color3.fromRGB(236, 236, 240),
		TextDim = Color3.fromRGB(132, 132, 140),
	},
	Slate = {
		Background = Color3.fromRGB(18, 20, 25),
		Surface = Color3.fromRGB(26, 29, 36),
		Text = Color3.fromRGB(232, 236, 244),
		TextDim = Color3.fromRGB(140, 148, 162),
	},
}

local THEME_ACCENTS = {
	{ "Aqua", "Darker", Color3.fromRGB(0, 255, 255) },
	{ "Sky", "Darker", Color3.fromRGB(88, 166, 255) },
	{ "Ocean", "Slate", Color3.fromRGB(64, 200, 224) },
	{ "Midnight", "Slate", Color3.fromRGB(96, 150, 255) },
	{ "Indigo", "Darker", Color3.fromRGB(120, 130, 255) },
	{ "Amethyst", "Darker", Color3.fromRGB(176, 128, 255) },
	{ "Orchid", "Dark", Color3.fromRGB(214, 130, 255) },
	{ "Rose", "Darker", Color3.fromRGB(255, 118, 160) },
	{ "Sakura", "Dark", Color3.fromRGB(255, 160, 190) },
	{ "Crimson", "Darker", Color3.fromRGB(255, 92, 92) },
	{ "Ember", "Dark", Color3.fromRGB(255, 122, 66) },
	{ "Amber", "Darker", Color3.fromRGB(255, 176, 66) },
	{ "Gold", "Dark", Color3.fromRGB(240, 205, 96) },
	{ "Lime", "Darker", Color3.fromRGB(164, 230, 84) },
	{ "Emerald", "Darker", Color3.fromRGB(72, 217, 150) },
	{ "Mint", "Slate", Color3.fromRGB(128, 240, 200) },
	{ "Teal", "Slate", Color3.fromRGB(64, 210, 196) },
	{ "Coral", "Dark", Color3.fromRGB(255, 127, 110) },
	{ "Peach", "Dark", Color3.fromRGB(255, 184, 140) },
	{ "Tangerine", "Darker", Color3.fromRGB(255, 149, 0) },
	{ "Lemon", "Dark", Color3.fromRGB(248, 236, 96) },
	{ "Forest", "Slate", Color3.fromRGB(96, 200, 110) },
	{ "Jade", "Darker", Color3.fromRGB(0, 200, 140) },
	{ "Azure", "Slate", Color3.fromRGB(0, 160, 255) },
	{ "Cobalt", "Slate", Color3.fromRGB(70, 110, 255) },
	{ "Violet", "Darker", Color3.fromRGB(150, 90, 255) },
	{ "Magenta", "Darker", Color3.fromRGB(255, 70, 200) },
	{ "Ruby", "Darker", Color3.fromRGB(230, 40, 80) },
	{ "Ice", "Slate", Color3.fromRGB(190, 235, 25
