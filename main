local args = table.pack(...)[1] or {}
local library = {
    title = args.title or "Window",
    foldername = args.foldername or "UILibrary",
    fileext = args.fileext or ".json",
    gamename = args.gamename or "universal",
    version = args.version or "1.0",
    tabSize = 0,
    maxSpots = 10,
    draggable = true,
    open = false,
    tabs = {},
    instances = {},
    connections = {},
    options = {},
    notifications = {},
    theme = {},
    ignored = {},
    prioritized = {},
    flags = {
        ["Menu Accent Color"] = Color3.new(0.7, 0.7, 0.7),
        ["Show Notifications"] = true
    }
}
--Locals
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TextService = game:GetService("TextService")
local HttpService = game:GetService("HttpService")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local CoreGui = game:GetService("CoreGui")
local Workspace = game:GetService("Workspace")
local camera = Workspace.CurrentCamera
local imageExtensions = { ".png", ".jpg", ".jpeg", ".tga", ".bmp" }
local dragging, dragOffset
local blacklistedKeys = { --add or remove keys if you find the need to
    Enum.KeyCode.Unknown,
    Enum.KeyCode.W,
    Enum.KeyCode.A,
    Enum.KeyCode.S,
    Enum.KeyCode.D,
    Enum.KeyCode.Slash,
    Enum.KeyCode.Tab,
    Enum.KeyCode.Escape
}
local whitelistedMouseinputs = { --add or remove mouse inputs if you find the need to
    Enum.UserInputType.MouseButton1,
    Enum.UserInputType.MouseButton2,
    Enum.UserInputType.MouseButton3
}
--Functions
function library.udim2ToVector2(value)
	return Vector2.new(
        camera.ViewportSize.X * value.X.Scale + value.X.Offset,
        camera.ViewportSize.Y * value.Y.Scale + value.Y.Offset
    )
end
function library.vector2ToUDim2(value)
	return UDim2.fromScale(
        value.X / camera.ViewportSize.X,
        value.Y / camera.ViewportSize.Y
    )
end
function library.converttotime(secs)
    secs = math.floor(secs)
    local mins = (secs - secs%60)/60
    secs = secs - mins*60
    local hours = (mins - mins%60)/60
    mins = mins - hours*60
    local days = (hours - hours%60)/60
    hours = hours - days*60
    local text = ""
    if days > 0 then
        text = text .. string.format("%02i", days)..":"
    end
    if hours > 0 then
        text = text .. string.format("%02i", hours)..":"
    end
    text = text .. string.format("%02i", mins) .. ":" .. string.format("%02i", secs)
    return text
end
function library.round(num, bracket)
    bracket = bracket or 1
    local a
    if typeof(num) == "Vector2" then
        a = Vector2.new(library.round(num.X), library.round(num.Y))
    elseif typeof(num) == "Color3" then
        return library.round(num.r * 255), library.round(num.g * 255), library.round(num.b * 255)
    else
        a = math.floor(num / bracket + (math.sign(num) * 0.5)) * bracket
        if a < 0 then
            a = a + bracket
        end
        return a
    end
    return a
end
function library:Create(class, properties)
    if not class then
        return
    end
    local properties = properties or {}
    local isdrawing = class == "Square" or class == "Line" or class == "Text" or class == "Quad" or class == "Circle" or class == "Triangle" or class == "Image"
    local inst = (isdrawing and Drawing or Instance).new(class)
    local newindex = getrawmetatable(inst).__newindex
    for property, value in next, properties do
        pcall(newindex, inst, property, value)
    end
    table.insert(self.instances, {
        object = inst,
        method = a
    })
    return inst
end
function library:AddConnection(connection, name, callback)
    callback = type(name) == "function" and name or callback
    connection = connection:connect(callback)
    if name ~= callback then
        self.connections[name] = connection
    else
        table.insert(self.connections, connection)
    end
    return connection
end
function library:Unload()
    for a, c in next, self.connections do
        c:Disconnect()
        self.connections[a] = nil
    end
    for _, i in next, self.instances do
        if i.method then
            pcall(function()
                i.object:Remove()
            end)
        else
            i.object:Destroy()
        end
    end
    for _, o in next, self.options do
        if o.type == "toggle" then
            pcall(function()
                o:SetState()
            end)
        end
    end
    for i,_ in next, self do
        self[i] = nil
    end
    getgenv().library = nil
end
function library:LoadConfig(config)
    if table.find(self:GetConfigs(), config) then
        local Read, Config = pcall(function()
            return HttpService:JSONDecode(readfile(self.foldername .. (self.gamename and ("/" .. self.gamename) or "") .. "/configs/" .. config .. self.fileext))
        end)
        Config = Read and Config or {}
        for _, option in next, self.options do
            if option.hasInit then
                if option.type ~= "button" and option.flag and not option.skipflag then
                    if option.type == "toggle" then
                        spawn(function()
                            option:SetState(Config[option.flag] == 1)
                        end)
                    elseif option.type == "color" then
                        if Config[option.flag] then
                            spawn(function()
                                option:SetColor(Config[option.flag])
                            end)
                            if option.trans then
                                spawn(function()
                                    option:SetTrans(Config[option.flag .. " Transparency"])
                                end)
                            end
                        end
                    elseif option.type == "bind" then
                        spawn(function()
                            option:SetKey(Config[option.flag])
                        end)
                    else
                        spawn(function()
                            option:SetValue(Config[option.flag])
                        end)
                    end
                end
            end
        end
    end
end
function library:SaveConfig(config)
    local Config = {}
    if table.find(self:GetConfigs(), config) then
        Config = HttpService:JSONDecode(readfile(self.foldername .. (self.gamename and ("/" .. self.gamename) or "") .. "/configs/" .. config .. self.fileext))
    end
    for _, option in next, self.options do
        if option.type ~= "button" and option.flag and not option.skipflag then
            if option.type == "toggle" then
                Config[option.flag] = option.state and 1 or 0
            elseif option.type == "color" then
                Config[option.flag] = {
                    option.color.r,
                    option.color.g,
                    option.color.b
                }
                if option.trans then
                    Config[option.flag .. " Transparency"] = option.trans
                end
            elseif option.type == "bind" then
                Config[option.flag] = option.key
            elseif option.type == "list" then
                Config[option.flag] = option.value
            else
                Config[option.flag] = option.value
            end
        end
    end
    writefile(self.foldername .. (self.gamename and ("/" .. self.gamename) or "") .. "/configs/" .. config .. self.fileext, HttpService:JSONEncode(Config))
end
function library:GetConfigs()
    if not isfolder(self.foldername) then
        makefolder(self.foldername)
        return {}
    end
    if self.gamename and not isfolder(self.foldername .. "/" .. self.gamename .. "/configs") then
        makefolder(self.foldername .. "/" .. self.gamename .. "/configs")
        return {}
    end
    local files = {}
    local a = 0
    for i, v in next, listfiles(self.foldername .. (self.gamename and ("/" .. self.gamename) .. "/configs" or "")) do
        if v:sub(#v - #self.fileext + 1, #v) == self.fileext then
            a = a + 1
            v = v:gsub((self.foldername .. (self.gamename and ("/" .. self.gamename) or "") .. "/configs") .. "\\", "")
            v = v:gsub(self.fileext, "")
            table.insert(files, a, v)
        end
    end
    return files
end
function library:GetBackgrounds()
    if not isfolder(self.foldername) then
        makefolder(self.foldername)
        return {}
    end
    if not isfolder(self.foldername .. "\\backgrounds") then
        makefolder(self.foldername .. "\\backgrounds")
        return {}
    end
    local images = { "Floral", "Flowers", "Circles", "Hearts", "Contour" }
    local a = 0
    for i, v in next, listfiles(self.foldername .. "\\backgrounds") do
        for _, ext in pairs(imageExtensions) do
            if v:sub(#v - #ext + 1, #v) == ext then
                a = a + 1
                v = v:gsub(self.foldername .. "\\backgrounds" .. "\\", "")
                v = v:gsub(ext, "")
                table.insert(images, a, v)
            end
        end
    end
    return images
end
function library:SetBackground(name)
    if not self.hasInit then
        return
    end
    if not name then return end
    local defaults = {
        Floral = "rbxassetid://5553946656",
        Flowers = "rbxassetid://6071575925",
        Circles = "rbxassetid://6071579801",
        Hearts = "rbxassetid://6073763717",
        Contour = "rbxassetid://2151741365"
    }
    if defaults[name] then self.main.Image = defaults[name] return end
    if not isfolder(self.foldername) or not isfolder(self.foldername .. "/backgrounds") then return false end
    local foundFile
    for i, v in next, listfiles(self.foldername .. "/backgrounds") do
        for _, ext in next, imageExtensions do
            if v:sub(#v - #ext + 1, #v) == ext then
                v = v:gsub(self.foldername .. "/backgrounds" .. "\\", "")
                if name == v:gsub(ext, "") then
                    foundFile = v
                end
            end
        end
    end
    if not foundFile then return false end
    self.main.Image = getsynasset(self.foldername .. "/backgrounds/" .. foundFile)
    return true
end
local spots = {}
for i = 1, library.maxSpots do
    spots[i] = false
end
function library:Notification(option)
    option.text = tostring(option.text or option.Text or "")
    option.time = tonumber(option.time or option.Time or 3)
    option.popuptime = tonumber(option.popuptime or option.PopupTime or 0.5)
    option.color = option.color or option.Color or Color3.new(0.85,0.85,0.85)
    if not self.flags["Show Notifications"] then
        return
    end
    local Spot
    repeat 
        task.wait()
        for i = 1, #spots do
            if spots[i] == false then
                Spot = i
                break
            end
        end
    until Spot
    option.main = self:Create("Frame", {
        Parent = self.base,
        BorderColor3 = Color3.fromRGB(80, 80, 80),
        Visible = true,
        BorderSizePixel = 0,
        ZIndex = 5,
        Size = UDim2.new(0, 0, 0, 20),
        AnchorPoint = Vector2.new(-1, 0),
        Position = UDim2.new(0, 10, 0, 50 + (Spot - 1) * 30),
        BackgroundColor3 = Color3.new(50, 50, 50)
    }) 
    self:Create("UIGradient", {
        Parent = option.main, 
        Rotation = 90,
        Color = ColorSequence.new({ 
            ColorSequenceKeypoint.new(0, Color3.fromRGB(50, 50, 50)), 
            ColorSequenceKeypoint.new(1, Color3.fromRGB(35, 35, 35)) 
        })
    })
    option.outline = self:Create("Frame", {
        Parent = option.main,
        ZIndex = 4,
        BorderSizePixel = 0,
        Visible = true,
        Size = UDim2.new(0, 0, 0, 22),
        BackgroundColor3 = Color3.fromRGB(20, 20, 20),
        Position = UDim2.fromOffset(-1, -1)
    })
    option.blackoutline = self:Create("Frame", {
        Parent = option.main,
        ZIndex = 3,
        BorderSizePixel = 0,
        BackgroundColor3 = Color3.new(),
        Visible = true,
        Size = UDim2.new(0, 0, 0, 0),
        Position = UDim2.fromOffset(-2, -2)
    })
    option.label = self:Create("TextLabel", {
        Parent = option.main,
        BackgroundTransparency = 1,
        Position = UDim2.new(),
        Size = UDim2.new(0, 0, 0, 20),
        Font = Enum.Font.Code,
        ZIndex = 6,
        Visible = true,
        RichText = true,
        Text = option.text,
        TextColor3 = option.color,
        TextSize = 0,
        TextTransparency = 1,
        TextStrokeTransparency = 0,
        TextXAlignment = Enum.TextXAlignment.Center
    })
    option.top = self:Create("Frame", {
        Parent = option.main,
        ZIndex = 6,
        BackgroundColor3 = self.flags["Menu Accent Color"],
        BorderSizePixel = 0,
        Visible = true,
        Size = UDim2.new(0, 0, 0, 1)
    })
    option.topline = self:Create("Frame", {
        Parent = option.main,
        ZIndex = 6,
        BorderSizePixel = 0,
        Visible = true,
        BackgroundColor3 = Color3.fromRGB(20, 20, 20),
        Size = UDim2.new(0, 0, 0, 1),
        Position = UDim2.new(0, 0, 0, 1)
    })
    table.insert(self.theme, option.top)
    local textSize = TextService:GetTextSize(option.label.Text, 14, option.label.Font, Vector2.new(9e9, 9e9))
    TweenService:Create(option.main, TweenInfo.new(option.popuptime), { Size = UDim2.new(0, textSize.X + 6, 0, 20) }):Play()
    TweenService:Create(option.label, TweenInfo.new(option.popuptime), { Size = UDim2.new(0, textSize.X + 6, 0, 20), TextTransparency = 0, TextSize = 14 }):Play()
    TweenService:Create(option.top, TweenInfo.new(option.popuptime), { Size = UDim2.new(0, textSize.X + 6, 0, 1) }):Play()
    TweenService:Create(option.topline, TweenInfo.new(option.popuptime), { Size = UDim2.new(0, textSize.X + 6, 0, 1) }):Play()
    TweenService:Create(option.outline, TweenInfo.new(option.popuptime), { Size = UDim2.new(0, textSize.X + 8, 0, 22) }):Play()
    TweenService:Create(option.blackoutline, TweenInfo.new(option.popuptime), { Size = UDim2.new(0, textSize.X + 10, 0, 24) }):Play()
    task.delay(option.time, function()
        TweenService:Create(option.main, TweenInfo.new(option.popuptime), { Size = UDim2.new(0, 0, 0, 20) }):Play()
        TweenService:Create(option.label, TweenInfo.new(option.popuptime), { Size = UDim2.new(0, 0, 0, 20), TextTransparency = 1, TextSize = 0 }):Play()
        TweenService:Create(option.top, TweenInfo.new(option.popuptime), { Size = UDim2.new(0, 0, 0, 1) }):Play()
        TweenService:Create(option.topline, TweenInfo.new(option.popuptime), { Size = UDim2.new(0, 0, 0, 1) }):Play()
        TweenService:Create(option.outline, TweenInfo.new(option.popuptime), { Size = UDim2.new(0, 0, 0, 22) }):Play()
        TweenService:Create(option.blackoutline, TweenInfo.new(option.popuptime), { Size = UDim2.new(0, 0, 0, 24) }):Play()
        task.wait(option.popuptime)
        table.remove(self.notifications, table.find(self.notifications, option))
        spots[Spot] = false
    end)
    spots[Spot] = true
    local notification = setmetatable({}, {
        __newindex = function(t, i, v)
            if i == "Text" then
                option.text = tostring(v)
                option.label.Text = option.Text
            end
            if i == "Color" then
                option.color = v
                option.label.TextColor = option.color
            end
            if i == "Visible" then
                option.main.Visible = option.visible
            end
        end,
        __index = function(t, i, v)
            if i == "Text" then
                return option.text
            end
            if i == "Color" then
                return option.label.TextColor
            end
            if i == "Visible" then
                return option.main.Visible
            end
        end
    })
    table.insert(self.notifications, notification)
    return notification
end
function library:AddBindList(option)
    if self.bindList then
        return
    end
    option.text = tostring(option.text or "")
    option.textcolor = option.textcolor or Color3.new(1,1,1)
    option.valuecolor = option.valuecolor or Color3.new(0.8,0.8,0.8)
    option.visible = (tostring(option.visible) or tostring(option.Visible) or "true") == "true"
    option.main = self:Create("Frame", {
        Parent = self.base,
        BorderColor3 = Color3.fromRGB(80, 80, 80),
        BorderSizePixel = 0,
        ZIndex = 5,
        Visible = option.visible,
        Size = UDim2.new(0, 160, 0, 18),
        AnchorPoint = Vector2.new(-1, 0),
        Position = UDim2.new(0, 10, 0, CurrentCamera.ViewportSize.Y / 2),
        BackgroundColor3 = Color3.new(50, 50, 50),
        Active = false
    }) 
    option.gradient = self:Create("UIGradient", {
        Parent = option.main, 
        Rotation = 90,
        Color = ColorSequence.new({ 
            ColorSequenceKeypoint.new(0, Color3.fromRGB(50, 50, 50)), 
            ColorSequenceKeypoint.new(1, Color3.fromRGB(35, 35, 35)) 
        })
    })
    option.outline = self:Create("Frame", {
        Parent = option.main,
        ZIndex = 4,
        BorderSizePixel = 0,
        BackgroundColor3 = Color3.fromRGB(20, 20, 20),
        Size = option.main.Size + UDim2.fromOffset(2, 2),
        Position = UDim2.fromOffset(-1, -1),
        Active = false
    })
    option.blackoutline = self:Create("Frame", {
        Parent = option.main,
        ZIndex = 3,
        BorderSizePixel = 0,
        BackgroundColor3 = Color3.new(),
        Size = option.main.Size + UDim2.fromOffset(4, 4),
        Position = UDim2.fromOffset(-2, -2),
        Active = false
    })
    option.label = self:Create("TextLabel", {
        Parent = option.main,
        BackgroundTransparency = 1,
        Position = UDim2.new(0, 3, 0, 0),
        Size = UDim2.new(0, 158, 0, 18),
        Font = Enum.Font.Code,
        ZIndex = 6,
        Text = option.text,
        TextColor3 = option.textcolor,
        TextSize = 13,
        TextStrokeTransparency = 0,
        TextXAlignment = Enum.TextXAlignment.Left,
        Active = false
    })
    option.top = self:Create("Frame", {
        Parent = option.main,
        ZIndex = 6,
        BackgroundColor3 = self.flags["Menu Accent Color"],
        BorderSizePixel = 0,
        Size = UDim2.new(0, 160, 0, 1),
        Active = false
    })
    table.insert(self.theme, option.top)
    option.topline = self:Create("Frame", {
        Parent = option.main,
        ZIndex = 6,
        BorderSizePixel = 0,
        BackgroundColor3 = Color3.fromRGB(20, 20, 20),
        Size = UDim2.new(0, 160, 0, 1),
        Position = UDim2.new(0, 0, 0, 1),
        Active = false
    })
    option.values = {}
    option.labels = {}
    local function refresh()
        for i,v in next, option.labels do
            v:Destroy()
        end
        option.labels = {}
        for i,v in next, option.values do
            if v and v[1] then
                table.insert(option.labels, self:Create("TextLabel", {
                    Parent = option.main,
                    BackgroundTransparency = 1,
                    Position = UDim2.new(0, 5, 0, 18 + #option.labels * 18),
                    Size = UDim2.new(0, TextService:GetTextSize(string.format("[%s] %s (%s)", tostring(v[3] or "none"), tostring(i), tostring(v[2])), 13, Enum.Font.Code, Vector2.new(9e9, 9e9)).X, 0, 13),
                    Font = Enum.Font.Code,
                    ZIndex = 6,
                    Text = string.format("[%s] %s (%s)", tostring(v[3] or "none"), tostring(i), tostring(v[2])),
                    TextColor3 = option.valuecolor,
                    TextSize = 13,
                    TextStrokeTransparency = 0,
                    TextXAlignment = Enum.TextXAlignment.Left,
                    Active = false
                }))
            end
        end
        local highestSize = 150
        for i,v in next, option.labels do
            local textSize = TextService:GetTextSize(v.Text, v.TextSize, v.Font, Vector2.new(9e9, 9e9))
            highestSize = math.max(highestSize, textSize.X + 10)
        end
        option.main.Size = UDim2.new(0, highestSize, 0, 18 + #option.labels * 18)
        option.main.Position = UDim2.new(0, 10, 0, CurrentCamera.ViewportSize.Y / 2 - option.main.Size.Y.Offset / 2)
        option.label.Size = UDim2.new(0, highestSize, 0, 18)
        option.top.Size = UDim2.new(0, highestSize, 0, 1)
        option.topline.Size = UDim2.new(0, highestSize, 0, 1)
        option.outline.Size = option.main.Size + UDim2.fromOffset(2, 2)
        option.blackoutline.Size = option.main.Size + UDim2.fromOffset(4, 4)
    end
    self.bindList = setmetatable({}, {
        __newindex = function(self, index, value)
            if string.lower(index) == "text" then
                option.label.Text = value
            elseif string.lower(index) == "textcolor" then
                option.label.TextColor3 = value
            elseif string.lower(index) == "valuecolor" then
                for i,v in next, option.labels do
                    v.TextColor3 = value
                end
            elseif string.lower(index) == "visible" then
                option.main.Visible = value
            else
                if option.values[index] then
                    option.values[index][1] = value
                    refresh()
                end
            end
        end,
        __index = function(self, index)
            if index == "AddValue" then
                return function(self, name, mode, default, defaultbind) 
                    option.values[name] = {
                        default, 
                        mode,
                        defaultbind
                    }
                    refresh()
                end
            elseif index == "UpdateBind" then
                return function(self, name, newbind)
                    if option.values[name] then
                        option.values[name][3] = newbind
                        refresh()
                    end
                end
            elseif index == "RemoveValue" then
                return function(self, name)
                    option.values[name] = nil
                    refresh()
                end
            elseif index == "Refresh" then
                return function(self)
                    refresh()
                end
            end
            return option.values[index] and option.values[index][1]
        end
    })
    return self.bindList
end
function library:AddWatermark(option)
    if self.watermark then
        return
    end
    option.visible = (tostring(option.visible) or tostring(option.Visible) or "true") == "true"
    option.text = tostring(option.text or "")
    option.color = option.color or option.Color or Color3.new(0.85,0.85,0.85)
    option.main = self:Create("Frame", {
        Parent = self.base,
        BorderColor3 = Color3.fromRGB(80, 80, 80),
        BorderSizePixel = 0,
        ZIndex = 5,
        Visible = option.visible,
        Size = UDim2.new(0, 0, 0, 0),
        AnchorPoint = Vector2.new(1, 0),
        Position = UDim2.new(1, -10, 0, 10),
        BackgroundColor3 = Color3.new(50, 50, 50),
        Active = false
    }) 
    option.gradient = self:Create("UIGradient", {
        Parent = option.main, 
        Rotation = 90,
        Color = ColorSequence.new({ 
            ColorSequenceKeypoint.new(0, Color3.fromRGB(50, 50, 50)), 
            ColorSequenceKeypoint.new(1, Color3.fromRGB(35, 35, 35)) 
        })
    })
    option.outline = self:Create("Frame", {
        Parent = option.main,
        ZIndex = 4,
        BorderSizePixel = 0,
        BackgroundColor3 = Color3.fromRGB(20, 20, 20),
        Position = UDim2.fromOffset(-1, -1),
        Active = false
    })
    option.blackoutline = self:Create("Frame", {
        Parent = option.main,
        ZIndex = 3,
        BorderSizePixel = 0,
        BackgroundColor3 = Color3.new(),
        Position = UDim2.fromOffset(-2, -2),
        Active = false
    })
    option.label = self:Create("TextLabel", {
        Parent = option.main,
        BackgroundTransparency = 1,
        Position = UDim2.new(0, 0, 0, 0),
        Size = UDim2.new(0, 238, 0, 20),
        Font = Enum.Font.Code,
        ZIndex = 6,
        Text = "",
        TextColor3 = option.color,
        TextSize = 14,
        TextStrokeTransparency = 0,
        TextXAlignment = Enum.TextXAlignment.Center,
        Active = false
    })
    option.top = self:Create("Frame", {
        Parent = option.main,
        ZIndex = 6,
        BackgroundColor3 = self.flags["Menu Accent Color"],
        BorderSizePixel = 0,
        Size = UDim2.new(0, 0, 0, 1),
        Active = false
    })
    option.topline = self:Create("Frame", {
        Parent = option.main,
        ZIndex = 6,
        BorderSizePixel = 0,
        BackgroundColor3 = Color3.fromRGB(20, 20, 20),
        Size = UDim2.new(0, 0, 0, 1),
        Position = UDim2.new(0, 0, 0, 1),
        Active = false
    })
    table.insert(self.theme, option.top)
    local textSize = TextService:GetTextSize(option.text, option.label.TextSize, option.label.Font, Vector2.new(9e9, 9e9))
    option.main.Size = UDim2.new(0, textSize.X + 8, 0, 20)    
    option.label.Size = UDim2.new(0, textSize.X + 8, 0, 20)
    option.top.Size = UDim2.new(0, textSize.X + 8, 0, 1)
    option.topline.Size = UDim2.new(0, textSize.X + 8, 0, 1)
    option.outline.Size = option.main.Size + UDim2.fromOffset(2, 2)
    option.blackoutline.Size = option.main.Size + UDim2.fromOffset(4, 4)
    local FPS, StartFPS, StartPING, RefreshTick = 0, 0, 60, 0, tick()
    self:AddConnection(RunService.RenderStepped, "fpsUpdater", function(deltaTime)
        StartFPS = 0.95 * StartFPS + 0.05 / deltaTime
        local newFPS = StartFPS - StartFPS % 1
        if newFPS ~= newFPS or newFPS == (1 / 0) or newFPS == (-1 / 0) then
            newFPS = 60
            StartFPS = 60
        end
        if newFPS then
            getgenv().current_fps = newFPS
        end
        if tick() - RefreshTick < 1 then
            return
        end
        RefreshTick = tick()
        if newFPS then
            FPS = newFPS
        end
    end)
    self:AddConnection(RunService.RenderStepped, "watermarkUpdater", function()
        local updatedText = option.text
        :gsub("{version}", tostring("v" .. self.version))
        :gsub("{username}", tostring(Players.LocalPlayer.Name))
        :gsub("{date}", tostring(os.date("%b %d %Y")))
        :gsub("{time}", tostring(os.date("%I:%M %p")))
        :gsub("{fps}", tostring(FPS))
        :gsub("{elapsedtime}", tostring(self.converttotime(elapsedTime())))
        local textSize = TextService:GetTextSize(updatedText, option.label.TextSize, option.label.Font, Vector2.new(9e9, 9e9))
        option.label.Text = updatedText
        option.main.Size = UDim2.new(0, textSize.X + 8, 0, 20)    
        option.label.Size = UDim2.new(0, textSize.X + 8, 0, 20)
        option.top.Size = UDim2.new(0, textSize.X + 8, 0, 1)
        option.topline.Size = UDim2.new(0, textSize.X + 8, 0, 1)
        option.outline.Size = option.main.Size + UDim2.fromOffset(2, 2)
        option.blackoutline.Size = option.main.Size + UDim2.fromOffset(4, 4)
    end)
    self.watermark = setmetatable({}, {
        __newindex = function(t, i, v)
            if i == "Text" then
                option.text = tostring(v)
            end
            if i == "Color" then
                option.color = v
                option.label.TextColor = option.color
            end
            if i == "Visible" then
                option.visible = v
                option.main.Visible = option.visible
            end
        end,
        __index = function(t, i)
            if i == "Text" then
                return option.text
            end
            if i == "Color" then
                return option.label.TextColor
            end
            if i == "Visible" then
                return option.main.Visible
            end
        end
    })
    return self.watermark
end
local function createLabel(option, parent)
    option.main = library:Create("TextLabel", {
        LayoutOrder = option.position,
        Position = UDim2.new(0, 6, 0, 0),
        Size = UDim2.new(1, -12, 0, 24),
        BackgroundTransparency = 1,
        TextSize = 15,
        Font = Enum.Font.Code,
        TextColor3 = Color3.new(1, 1, 1),
        TextXAlignment = Enum.TextXAlignment.Left,
        TextYAlignment = Enum.TextYAlignment.Top,
        TextWrapped = true,
        Parent = parent
    })
    setmetatable(option, {
        __newindex = function(t, i, v)
            if i == "Text" then
                option.main.Text = tostring(v)
                option.main.Size = UDim2.new(1, -12, 0, TextService:GetTextSize(option.main.Text, 15, Enum.Font.Code, Vector2.new(option.main.AbsoluteSize.X, 9e9)).Y + 6)
            end
        end
    })
    option.Text = option.text
end
local function createDivider(option, parent)
    option.hasInit = true
    option.main = library:Create("Frame", {
        LayoutOrder = option.position,
        Size = UDim2.new(1, 0, 0, 18),
        BackgroundTransparency = 1,
        Parent = parent
    })
    library:Create("Frame", {
        AnchorPoint = Vector2.new(0.5, 0.5),
        Position = UDim2.new(0.5, 0, 0.5, 0),
        Size = UDim2.new(1, -24, 0, 1),
        BackgroundColor3 = Color3.fromRGB(71, 69, 71),
        BorderColor3 = Color3.new(),
        Parent = option.main
    })
    option.title = library:Create("TextLabel", {
        AnchorPoint = Vector2.new(0.5, 0.5),
        Position = UDim2.new(0.5, 0, 0.5, 0),
        BackgroundColor3 = Color3.fromRGB(30, 30, 30),
        BorderSizePixel = 0,
        TextColor3 =  Color3.new(1, 1, 1),
        TextSize = 15,
        Font = Enum.Font.Code,
        TextXAlignment = Enum.TextXAlignment.Center,
        Parent = option.main
    })
    setmetatable(option, {
        __newindex = function(t, i, v)
            if i == "Text" then
                if v then
                    option.title.Text = tostring(v)
                    option.title.Size = UDim2.new(0, TextService:GetTextSize(option.title.Text, 15, Enum.Font.Code, Vector2.new(9e9, 9e9)).X + 12, 0, 20)
                    option.main.Size = UDim2.new(1, 0, 0, 18)
                else
                    option.title.Text = ""
                    option.title.Size = UDim2.new()
                    option.main.Size = UDim2.new(1, 0, 0, 6)
                end
            end
        end
    })
    option.Text = option.text
end
local function createToggle(option, parent)
    option.hasInit = true
    option.main = library:Create("Frame", {
        LayoutOrder = option.position,
        Size = UDim2.new(1, 0, 0, 20),
        BackgroundTransparency = 1,
        Parent = parent
    })
    local tickbox
    local tickboxOverlay
    if option.style then
        tickbox = library:Create("ImageLabel", {
            Position = UDim2.new(0, 6, 0, 4),
            Size = UDim2.new(0, 12, 0, 12),
            BackgroundTransparency = 1,
            Image = "rbxassetid://3570695787",
            ImageColor3 = Color3.new(),
            Parent = option.main
        })
        library:Create("ImageLabel", {
            AnchorPoint = Vector2.new(0.5, 0.5),
            Position = UDim2.new(0.5, 0, 0.5, 0),
            Size = UDim2.new(1, -2, 1, -2),
            BackgroundTransparency = 1,
            Image = "rbxassetid://3570695787",
            ImageColor3 = Color3.fromRGB(60, 60, 60),
            Parent = tickbox
        })
        library:Create("ImageLabel", {
            AnchorPoint = Vector2.new(0.5, 0.5),
            Position = UDim2.new(0.5, 0, 0.5, 0),
            Size = UDim2.new(1, -6, 1, -6),
            BackgroundTransparency = 1,
            Image = "rbxassetid://3570695787",
            ImageColor3 = Color3.fromRGB(40, 40, 40),
            Parent = tickbox
        })
        tickboxOverlay = library:Create("ImageLabel", {
            AnchorPoint = Vector2.new(0.5, 0.5),
            Position = UDim2.new(0.5, 0, 0.5, 0),
            Size = UDim2.new(1, -6, 1, -6),
            BackgroundTransparency = 1,
            Image = "rbxassetid://3570695787",
            ImageColor3 = library.flags["Menu Accent Color"],
            Visible = false,
            Parent = tickbox
        })
        library:Create("ImageLabel", {
            AnchorPoint = Vector2.new(0.5, 0.5),
            Position = UDim2.new(0.5, 0, 0.5, 0),
            Size = UDim2.new(1, 0, 1, 0),
            BackgroundTransparency = 1,
            Image = "rbxassetid://5941353943",
            ImageTransparency = 0.6,
            Parent = tickbox
        })
        table.insert(library.theme, tickboxOverlay)
    else
        tickbox = library:Create("Frame", {
            Position = UDim2.new(0, 6, 0, 4),
            Size = UDim2.new(0, 12, 0, 12),
            BackgroundColor3 = library.flags["Menu Accent Color"],
            BorderColor3 = Color3.new(),
            Parent = option.main
        })
        tickboxOverlay = library:Create("ImageLabel", {
            Size = UDim2.new(1, 0, 1, 0),
            BackgroundTransparency = 0,
            BackgroundColor3 = Color3.fromRGB(50, 50, 50),
            BorderColor3 = Color3.new(),
            Image = "rbxassetid://4155801252",
            ImageTransparency = 0.6,
            ImageColor3 = Color3.new(),
            Parent = tickbox
        })
        library:Create("ImageLabel", {
            Size = UDim2.new(1, 0, 1, 0),
            BackgroundTransparency = 1,
            Image = "rbxassetid://2592362371",
            ImageColor3 = Color3.fromRGB(60, 60, 60),
            ScaleType = Enum.ScaleType.Slice,
            SliceCenter = Rect.new(2, 2, 62, 62),
            Parent = tickbox
        })
        library:Create("ImageLabel", {
            Size = UDim2.new(1, -2, 1, -2),
            Position = UDim2.new(0, 1, 0, 1),
            BackgroundTransparency = 1,
            Image = "rbxassetid://2592362371",
            ImageColor3 = Color3.new(),
            ScaleType = Enum.ScaleType.Slice,
            SliceCenter = Rect.new(2, 2, 62, 62),
            Parent = tickbox
        })
        table.insert(library.theme, tickbox)
    end
    option.interest = library:Create("Frame", {
        Position = UDim2.new(0, 0, 0, 0),
        Size = UDim2.new(1, 0, 0, 20),
        BackgroundTransparency = 1,
        Parent = option.main
    })
    option.title = library:Create("TextLabel", {
        Position = UDim2.new(0, 24, 0, 0),
        Size = UDim2.new(1, 0, 1, 0),
        BackgroundTransparency = 1,
        Text = option.text,
        TextColor3 = Color3.fromRGB(180, 180, 180),
        TextSize = 15,
        Font = Enum.Font.Code,
        TextXAlignment = Enum.TextXAlignment.Left,
        Parent = option.interest
    })
    library:AddConnection(option.interest.InputBegan, function(input)
        if input.UserInputType.Name == "MouseButton1" then
            option:SetState(not option.state)
        end
        if input.UserInputType.Name == "MouseMovement" then
            if not library.warning and not library.slider then
                if option.style then
                    tickbox.ImageColor3 = library.flags["Menu Accent Color"]
                else
                    tickbox.BorderColor3 = library.flags["Menu Accent Color"]
                    tickboxOverlay.BorderColor3 = library.flags["Menu Accent Color"]
                end
            end
            if option.tip then
                library.tooltip.Text = option.tip
                local size = TextService:GetTextSize(option.tip, 15, Enum.Font.Code, Vector2.new(9e9, 9e9))
                library.tooltip.Size = UDim2.new(0, size.X, 0, size.Y)
            end
        end
    end)
    library:AddConnection(option.interest.InputChanged, function(input)
        if input.UserInputType.Name == "MouseMovement" then
            if option.tip then
                library.tooltip.Position = UDim2.new(0, input.Position.X + 26, 0, input.Position.Y + 36)
            end
        end
    end)
    library:AddConnection(option.interest.InputEnded, function(input)
        if input.UserInputType.Name == "MouseMovement" then
            if option.style then
                tickbox.ImageColor3 = Color3.new()
            else
                tickbox.BorderColor3 = Color3.new()
                tickboxOverlay.BorderColor3 = Color3.new()
            end
            library.tooltip.Position = UDim2.new(2)
        end
    end)
    function option:SetState(state, nocallback)
        state = typeof(state) == "boolean" and state
        state = state or false
        library.flags[self.flag] = state
        self.state = state
        option.title.TextColor3 = state and Color3.fromRGB(210, 210, 210) or Color3.fromRGB(160, 160, 160)
        if option.style then
            tickboxOverlay.Visible = state
        else
            tickboxOverlay.BackgroundTransparency = state and 1 or 0
        end
        if not nocallback then
            self.callback(state)
        end
    end
    task.delay(1, function()
        if library then
            option:SetState(option.state)
        end
    end)
    setmetatable(option, {
        __newindex = function(t, i, v)
            if i == "Text" then
                option.title.Text = tostring(v)
            end
        end
    })
end

local function createButton(option, parent)
    option.hasInit = true
    option.main = library:Create("Frame", {
        LayoutOrder = option.position,
        Size = UDim2.new(1, 0, 0, 26),
        BackgroundTransparency = 1,
        Parent = parent
    })
    option.title = library:Create("TextLabel", {
        AnchorPoint = Vector2.new(0.5, 1),
        Position = UDim2.new(0.5, 0, 1, -5),
        Size = UDim2.new(1, -12, 0, 18),
        BackgroundColor3 = Color3.fromRGB(50, 50, 50),
        BorderColor3 = Color3.new(),
        Text = option.text,
        TextColor3 = option.textcolor or Color3.new(1, 1, 1),
        TextSize = 15,
        Font = Enum.Font.Code,
        Parent = option.main
    })
    library:Create("ImageLabel", {
        Size = UDim2.new(1, 0, 1, 0),
        BackgroundTransparency = 1,
        Image = "rbxassetid://2592362371",
        ImageColor3 = Color3.fromRGB(60, 60, 60),
        ScaleType = Enum.ScaleType.Slice,
        SliceCenter = Rect.new(2, 2, 62, 62),
        Parent = option.title
    })
    library:Create("ImageLabel", {
        Size = UDim2.new(1, -2, 1, -2),
        Position = UDim2.new(0, 1, 0, 1),
        BackgroundTransparency = 1,
        Image = "rbxassetid://2592362371",
        ImageColor3 = Color3.new(),
        ScaleType = Enum.ScaleType.Slice,
        SliceCenter = Rect.new(2, 2, 62, 62),
        Parent = option.title
    })
    library:Create("UIGradient", {
        Color = ColorSequence.new({
            ColorSequenceKeypoint.new(0, Color3.fromRGB(180, 180, 180)),
            ColorSequenceKeypoint.new(1, Color3.fromRGB(253, 253, 253)),
        }),
        Rotation = -90,
        Parent = option.title
    })
    library:AddConnection(option.title.InputBegan, function(input)
        if input.UserInputType.Name == "MouseButton1" then
            option.callback()
            if library then
                library.flags[option.flag] = true
            end
            if option.tip then
                library.tooltip.Text = option.tip
                local size = TextService:GetTextSize(option.tip, 15, Enum.Font.Code, Vector2.new(9e9, 9e9))
                library.tooltip.Size = UDim2.new(0, size.X, 0, size.Y)
            end
        end
        if input.UserInputType.Name == "MouseMovement" then
            if not library.warning and not library.slider then
                option.title.BorderColor3 = library.flags["Menu Accent Color"]
            end
        end
    end)
    library:AddConnection(option.title.InputChanged, function(input)
        if input.UserInputType.Name == "MouseMovement" then
            if option.tip then
                library.tooltip.Position = UDim2.new(0, input.Position.X + 26, 0, input.Position.Y + 36)
            end
        end
    end)
    library:AddConnection(option.title.InputEnded, function(input)
        if input.UserInputType.Name == "MouseMovement" then
            option.title.BorderColor3 = Color3.new()
            library.tooltip.Position = UDim2.new(2)
        end
    end)
end

local function createBind(option, parent)
    option.hasInit = true
    local binding
    local holding
    local Loop
    if option.sub then
        option.main = option:getMain()
    else
        option.main = option.main or library:Create("Frame", {
            LayoutOrder = option.position,
            Size = UDim2.new(1, 0, 0, 20),
            BackgroundTransparency = 1,
            Parent = parent
        })
        library:Create("TextLabel", {
            Position = UDim2.new(0, 6, 0, 0),
            Size = UDim2.new(1, -12, 1, 0),
            BackgroundTransparency = 1,
            Text = option.text,
            TextSize = 15,
            Font = Enum.Font.Code,
            TextColor3 = Color3.fromRGB(210, 210, 210),
            TextXAlignment = Enum.TextXAlignment.Left,
            Parent = option.main
        })
    end
    local bindinput = library:Create(option.sub and "TextButton" or "TextLabel", {
        Position = UDim2.new(1, -6 - (option.subpos or 0), 0, option.sub and 2 or 3),
        SizeConstraint = Enum.SizeConstraint.RelativeYY,
        BackgroundColor3 = Color3.fromRGB(30, 30, 30),
        BorderSizePixel = 0,
        TextSize = 15,
        Font = Enum.Font.Code,
        TextColor3 = Color3.fromRGB(160, 160, 160),
        TextXAlignment = Enum.TextXAlignment.Right,
        Parent = option.main
    })
    if option.sub then
        bindinput.AutoButtonColor = false
    end
    local interest = option.sub and bindinput or option.main
    local inContact
    library:AddConnection(interest.InputEnded, function(input)
        if input.UserInputType.Name == "MouseButton1" then
            binding = true
            bindinput.Text = "[...]"
            bindinput.Size = UDim2.new(0, -TextService:GetTextSize(bindinput.Text, 16, Enum.Font.Code, Vector2.new(9e9, 9e9)).X, 0, 16)
            bindinput.TextColor3 = library.flags["Menu Accent Color"]
        end
    end)
    library:AddConnection(UserInputService.InputBegan, function(input)
        if UserInputService:GetFocusedTextBox() then
            return
        end
        if binding then
            local key = (table.find(whitelistedMouseinputs, input.UserInputType) and not option.nomouse) and input.UserInputType
            option:SetKey(key or (not table.find(blacklistedKeys, input.KeyCode)) and input.KeyCode)
        else
            if (input.KeyCode.Name == option.key or input.UserInputType.Name == option.key) and not binding then
                if option.mode == "toggle" then
                    library.flags[option.flag] = not library.flags[option.flag]
                    option.callback(library.flags[option.flag], 0)
                else
                    library.flags[option.flag] = true
                    if Loop then
                        Loop:Disconnect()
                        option.callback(true, 0)
                    end
                    Loop = library:AddConnection(RunService.RenderStepped, function(step)
                        if not UserInputService:GetFocusedTextBox() then
                            option.callback(nil, step)
                        end
                    end)
                end
            end
        end
    end)
    library:AddConnection(UserInputService.InputEnded, function(input)
        if option.key ~= "none" then
            if input.KeyCode.Name == option.key or input.UserInputType.Name == option.key then
                if Loop then
                    Loop:Disconnect()
                    library.flags[option.flag] = false
                    option.callback(true, 0)
                end
            end
        end
    end)
    function option:SetKey(key)
        binding = false
        bindinput.TextColor3 = Color3.fromRGB(160, 160, 160)
        if Loop then
            Loop:Disconnect()
            library.flags[option.flag] = false
            option.callback(true, 0)
        end
        self.key = (key and key.Name) or key or self.key
        if self.key == "Backspace" then
            self.key = "none"
            bindinput.Text = "[NONE]"
        else
            local a = self.key
            if self.key:match"Mouse" then
                a = self.key:gsub("Button", ""):gsub("Mouse", "M")
            elseif self.key:match"Shift" or self.key:match"Alt" or self.key:match"Control" then
                a = self.key:gsub("Left", "L"):gsub("Right", "R")
            end
            bindinput.Text = "[" .. a:gsub("Control", "CTRL"):upper() .. "]"
        end
        bindinput.Size = UDim2.new(0, -TextService:GetTextSize(bindinput.Text, 16, Enum.Font.Code, Vector2.new(9e9, 9e9)).X, 0, 16)
        option.keycallback(self.key)
    end
    option:SetKey()
end

local function createSlider(option, parent)
    option.hasInit = true
    if option.sub then
        option.main = option:getMain()
        option.main.Size = UDim2.new(1, 0, 0, 42)
    else
        option.main = library:Create("Frame", {
            LayoutOrder = option.position,
            Size = UDim2.new(1, 0, 0, option.textpos and 24 or 40),
            BackgroundTransparency = 1,
            Parent = parent
        })
    end
    option.slider = library:Create("Frame", {
        Position = UDim2.new(0, 6, 0, (option.sub and 22 or option.textpos and 4 or 20)),
        Size = UDim2.new(1, -12, 0, 16),
        BackgroundColor3 = Color3.fromRGB(50, 50, 50),
        BorderColor3 = Color3.new(),
        Parent = option.main
    })
    library:Create("ImageLabel", {
        Size = UDim2.new(1, 0, 1, 0),
        BackgroundTransparency = 1,
        Image = "rbxassetid://2454009026",
        ImageColor3 = Color3.new(),
        ImageTransparency = 0.8,
        Parent = option.slider
    })
    option.fill = library:Create("Frame", {
        BackgroundColor3 = library.flags["Menu Accent Color"],
        BorderSizePixel = 0,
        Parent = option.slider
    })
    library:Create("ImageLabel", {
        Size = UDim2.new(1, 0, 1, 0),
        BackgroundTransparency = 1,
        Image = "rbxassetid://2592362371",
        ImageColor3 = Color3.fromRGB(60, 60, 60),
        ScaleType = Enum.ScaleType.Slice,
        SliceCenter = Rect.new(2, 2, 62, 62),
        Parent = option.slider
    })
    library:Create("ImageLabel", {
        Size = UDim2.new(1, -2, 1, -2),
        Position = UDim2.new(0, 1, 0, 1),
        BackgroundTransparency = 1,
        Image = "rbxassetid://2592362371",
        ImageColor3 = Color3.new(),
        ScaleType = Enum.ScaleType.Slice,
        SliceCenter = Rect.new(2, 2, 62, 62),
        Parent = option.slider
    })
    option.title = library:Create("TextBox", {
        Position = UDim2.new((option.sub or option.textpos) and 0.5 or 0, (option.sub or option.textpos) and 0 or 6, 0, 0),
        Size = UDim2.new(0, 0, 0, (option.sub or option.textpos) and 14 or 18),
        BackgroundTransparency = 1,
        Text = (option.text == "nil" and "" or option.text .. ": ") .. option.value .. option.suffix,
        TextSize = (option.sub or option.textpos) and 14 or 15,
        Font = Enum.Font.Code,
        TextColor3 = Color3.fromRGB(210, 210, 210),
        TextXAlignment = Enum.TextXAlignment[(option.sub or option.textpos) and "Center" or "Left"],
        Parent = (option.sub or option.textpos) and option.slider or option.main
    })
    table.insert(library.theme, option.fill)
    library:Create("UIGradient", {
        Color = ColorSequence.new({
            ColorSequenceKeypoint.new(0, Color3.fromRGB(115, 115, 115)),
            ColorSequenceKeypoint.new(1, Color3.new(1, 1, 1)),
        }),
        Rotation = -90,
        Parent = option.fill
    })
    if option.min >= 0 then
        option.fill.Size = UDim2.new((option.value - option.min) / (option.max - option.min), 0, 1, 0)
    else
        option.fill.Position = UDim2.new((0 - option.min) / (option.max - option.min), 0, 0, 0)
        option.fill.Size = UDim2.new(option.value / (option.max - option.min), 0, 1, 0)
    end
    local manualInput
    library:AddConnection(option.title.Focused, function()
        if not manualInput then
            option.title:ReleaseFocus()
            option.title.Text = (option.text == "nil" and "" or option.text .. ": ") .. option.value .. option.suffix
        end
    end)
    library:AddConnection(option.title.FocusLost, function()
        option.slider.BorderColor3 = Color3.new()
        if manualInput then
            if tonumber(option.title.Text) then
                option:SetValue(tonumber(option.title.Text))
            else
                option.title.Text = (option.text == "nil" and "" or option.text .. ": ") .. option.value .. option.suffix
            end
        end
        manualInput = false
    end)

    local interest = (option.sub or option.textpos) and option.slider or option.main
    library:AddConnection(interest.InputBegan, function(input)
        if input.UserInputType.Name == "MouseButton1" then
            if UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) or UserInputService:IsKeyDown(Enum.KeyCode.RightControl) then
                manualInput = true
                option.title:CaptureFocus()
            else
                library.slider = option
                option.slider.BorderColor3 = library.flags["Menu Accent Color"]
                option:SetValue(option.min + ((input.Position.X - option.slider.AbsolutePosition.X) / option.slider.AbsoluteSize.X) * (option.max - option.min))
            end
        end
        if input.UserInputType.Name == "MouseMovement" then
            if not library.warning and not library.slider then
                option.slider.BorderColor3 = library.flags["Menu Accent Color"]
            end
            if option.tip then
                library.tooltip.Text = option.tip
                local size = TextService:GetTextSize(option.tip, 15, Enum.Font.Code, Vector2.new(9e9, 9e9))
                library.tooltip.Size = UDim2.new(0, size.X, 0, size.Y)
            end
        end
    end)
    library:AddConnection(interest.InputChanged, function(input)
        if input.UserInputType.Name == "MouseMovement" then
            if option.tip then
                library.tooltip.Position = UDim2.new(0, input.Position.X + 26, 0, input.Position.Y + 36)
            end
        end
    end)
    library:AddConnection(interest.InputEnded, function(input)
        if input.UserInputType.Name == "MouseButton1" then
            library.tooltip.Position = UDim2.new(2)
            library.slider = nil
            if option ~= library.slider then
                option.slider.BorderColor3 = Color3.new()
                    --option.fill.BorderColor3 = Color3.new()
            end
        end
    end)
    library:AddConnection(RunService.RenderStepped, function()
        if library.slider == option then
            local position = UserInputService:GetMouseLocation()
            option:SetValue(option.min + ((position.X - option.slider.AbsolutePosition.X) / option.slider.AbsoluteSize.X) * (option.max - option.min))
        end
    end)
    function option:SetValue(value, nocallback)
        if typeof(value) ~= "number" then
            value = 0
        end
        value = library.round(value, option.float)
        value = math.clamp(value, self.min, self.max)
        if self.min >= 0 then
            option.fill.Size = UDim2.new((value - self.min) / (self.max - self.min), 0, 1, 0)
        else
            option.fill.Position = UDim2.new((0 - self.min) / (self.max - self.min), 0, 0, 0)
            option.fill.Size = UDim2.new(value / (self.max - self.min), 0, 1, 0)
        end
        library.flags[self.flag] = value
        self.value = value
        local reachedEnd = false
        if option.endname and self.value == self.max then
            option.title.Text = (option.text == "nil" and "" or option.text .. ": ") .. option.endname
            reachedEnd = true
        else
            option.title.Text = (option.text == "nil" and "" or option.text .. ": ") .. option.value .. option.suffix
        end
        if not nocallback then
            self.callback(value, reachedEnd)
        end
    end
    task.delay(1, function()
        if library then
            option:SetValue(option.value)
        end
    end)
end

local function createList(option, parent)
    option.hasInit = true
    if option.sub then
        option.main = option:getMain()
        option.main.Size = UDim2.new(1, 0, 0, 48)
    else
        option.main = library:Create("Frame", {
            LayoutOrder = option.position,
            Size = UDim2.new(1, 0, 0, option.text == "nil" and 30 or 48),
            BackgroundTransparency = 1,
            Parent = parent
        })
        if option.text ~= "nil" then
            library:Create("TextLabel", {
                Position = UDim2.new(0, 6, 0, 0),
                Size = UDim2.new(1, -12, 0, 18),
                BackgroundTransparency = 1,
                Text = option.text,
                TextSize = 15,
                Font = Enum.Font.Code,
                TextColor3 = Color3.fromRGB(210, 210, 210),
                TextXAlignment = Enum.TextXAlignment.Left,
                Parent = option.main
            })
        end
    end
    local function getMultiText()
        local s = ""
        for _, value in next, option.values do
            s = s .. ((table.find(option.value, value) or option.value[value]) and (tostring(value) .. ", ") or "")
        end
        local t = string.sub(s, 1, #s - 2)
        return t
    end
    option.listvalue = library:Create("TextLabel", {
        Position = UDim2.new(0, 6, 0, (option.text == "nil" and not option.sub) and 4 or 22),
        Size = UDim2.new(1, -12, 0, 22),
        BackgroundColor3 = Color3.fromRGB(50, 50, 50),
        BorderColor3 = Color3.new(),
        Text = " " .. (typeof(option.value) == "string" and option.value or getMultiText()),
        TextSize = 15,
        Font = Enum.Font.Code,
        TextColor3 = Color3.new(1, 1, 1),
        TextXAlignment = Enum.TextXAlignment.Left,
        TextTruncate = Enum.TextTruncate.AtEnd,
        Parent = option.main
    })
    library:Create("ImageLabel", {
        Size = UDim2.new(1, 0, 1, 0),
        BackgroundTransparency = 1,
        Image = "rbxassetid://2454009026",
        ImageColor3 = Color3.new(),
        ImageTransparency = 0.8,
        Parent = option.listvalue
    })
    library:Create("ImageLabel", {
        Size = UDim2.new(1, 0, 1, 0),
        BackgroundTransparency = 1,
        Image = "rbxassetid://2592362371",
        ImageColor3 = Color3.fromRGB(60, 60, 60),
        ScaleType = Enum.ScaleType.Slice,
        SliceCenter = Rect.new(2, 2, 62, 62),
        Parent = option.listvalue
    })
    library:Create("ImageLabel", {
        Size = UDim2.new(1, -2, 1, -2),
        Position = UDim2.new(0, 1, 0, 1),
        BackgroundTransparency = 1,
        Image = "rbxassetid://2592362371",
        ImageColor3 = Color3.new(),
        ScaleType = Enum.ScaleType.Slice,
        SliceCenter = Rect.new(2, 2, 62, 62),
        Parent = option.listvalue
    })
    option.arrow = library:Create("ImageLabel", {
        Position = UDim2.new(1, -16, 0, 7),
        Size = UDim2.new(0, 8, 0, 8),
        Rotation = 90,
        BackgroundTransparency = 1,
        Image = "rbxassetid://4918373417",
        ImageColor3 = Color3.new(1, 1, 1),
        ScaleType = Enum.ScaleType.Fit,
        ImageTransparency = 0.4,
        Parent = option.listvalue
    })
    option.holder = library:Create("TextButton", {
        ZIndex = 4,
        BackgroundColor3 = Color3.fromRGB(40, 40, 40),
        BorderColor3 = Color3.new(),
        Text = "",
        AutoButtonColor = false,
        Visible = false,
        Parent = library.base
    })
    option.content = library:Create("ScrollingFrame", {
        ZIndex = 4,
        Size = UDim2.new(1, 0, 1, 0),
        BackgroundTransparency = 1,
        BorderSizePixel = 0,
        ScrollBarImageColor3 = Color3.new(),
        ScrollBarThickness = 3,
        ScrollingDirection = Enum.ScrollingDirection.Y,
        VerticalScrollBarInset = Enum.ScrollBarInset.Always,
        TopImage = "rbxasset://textures/ui/Scroll/scroll-middle.png",
        BottomImage = "rbxasset://textures/ui/Scroll/scroll-middle.png",
        Parent = option.holder
    })
    library:Create("ImageLabel", {
        ZIndex = 4,
        Size = UDim2.new(1, 0, 1, 0),
        BackgroundTransparency = 1,
        Image = "rbxassetid://2592362371",
        ImageColor3 = Color3.fromRGB(60, 60, 60),
        ScaleType = Enum.ScaleType.Slice,
        SliceCenter = Rect.new(2, 2, 62, 62),
        Parent = option.holder
    })
    library:Create("ImageLabel", {
        ZIndex = 4,
        Size = UDim2.new(1, -2, 1, -2),
        Position = UDim2.new(0, 1, 0, 1),
        BackgroundTransparency = 1,
        Image = "rbxassetid://2592362371",
        ImageColor3 = Color3.new(),
        ScaleType = Enum.ScaleType.Slice,
        SliceCenter = Rect.new(2, 2, 62, 62),
        Parent = option.holder
    })
    local layout = library:Create("UIListLayout", {
        Padding = UDim.new(0, 2),
        Parent = option.content
    })
    library:Create("UIPadding", {
        PaddingTop = UDim.new(0, 4),
        PaddingLeft = UDim.new(0, 4),
        Parent = option.content
    })
    local valueCount = 0
    library:AddConnection(layout.Changed, function()
        option.holder.Size = UDim2.new(0, option.listvalue.AbsoluteSize.X, 0, 8 + (valueCount > option.max and (-2 + (option.max * 22)) or layout.AbsoluteContentSize.Y))
        option.content.CanvasSize = UDim2.new(0, 0, 0, 8 + layout.AbsoluteContentSize.Y)
    end)
    local interest = option.sub and option.listvalue or option.main
    library:AddConnection(option.listvalue.InputBegan, function(input)
        if input.UserInputType.Name == "MouseButton1" then
            if library.popup == option then
                library.popup:Close()
                return
            end
            if library.popup then
                library.popup:Close()
            end
            option.arrow.Rotation = -90
            option.open = true
            option.holder.Visible = true
            local pos = option.main.AbsolutePosition
            option.holder.Position = UDim2.new(0, pos.X + 6, 0, pos.Y + ((option.text == "nil" and not option.sub) and 66 or 84))
            library.popup = option
            option.listvalue.BorderColor3 = library.flags["Menu Accent Color"]
        end
        if input.UserInputType.Name == "MouseMovement" then
            if not library.warning and not library.slider then
                option.listvalue.BorderColor3 = library.flags["Menu Accent Color"]
            end
        end
    end)
    library:AddConnection(option.listvalue.InputEnded, function(input)
        if input.UserInputType.Name == "MouseMovement" then
            if not option.open then
                option.listvalue.BorderColor3 = Color3.new()
            end
        end
    end)
    library:AddConnection(interest.InputBegan, function(input)
        if input.UserInputType.Name == "MouseMovement" then
            if option.tip then
                library.tooltip.Text = option.tip
                local size = TextService:GetTextSize(option.tip, 15, Enum.Font.Code, Vector2.new(9e9, 9e9))
                library.tooltip.Size = UDim2.new(0, size.X, 0, size.Y)
            end
        end
    end)
    library:AddConnection(interest.InputChanged, function(input)
        if input.UserInputType.Name == "MouseMovement" then
            if option.tip then
                library.tooltip.Position = UDim2.new(0, input.Position.X + 26, 0, input.Position.Y + 36)
            end
        end
    end)
    library:AddConnection(interest.InputEnded, function(input)
        if input.UserInputType.Name == "MouseMovement" then
            library.tooltip.Position = UDim2.new(2)
        end
    end)
    local selected
    function option:AddValue(value, state)
        if self.labels[value] then
            return
        end
        valueCount = valueCount + 1
        if self.multiselect then
            self.values[value] = state
        else
            if not table.find(self.values, value) then
                table.insert(self.values, value)
            end
        end
        local label = library:Create("TextLabel", {
            ZIndex = 4,
            Size = UDim2.new(1, 0, 0, 20),
            BackgroundTransparency = 1,
            Text = value,
            TextSize = 15,
            Font = Enum.Font.Code,
            TextTransparency = self.multiselect and (self.value[value] and 1 or 0) or self.value == value and 1 or 0,
            TextColor3 = Color3.fromRGB(210, 210, 210),
            TextXAlignment = Enum.TextXAlignment.Left,
            Parent = option.content
        })
        self.labels[value] = label
        local labelOverlay = library:Create("TextLabel", {
            ZIndex = 4,
            Size = UDim2.new(1, 0, 1, 0),
            BackgroundTransparency = 0.8,
            Text = " " .. value,
            TextSize = 15,
            Font = Enum.Font.Code,
            TextColor3 = library.flags["Menu Accent Color"],
            TextXAlignment = Enum.TextXAlignment.Left,
            Visible = self.multiselect and self.value[value] or self.value == value,
            Parent = label
        })
        selected = selected or self.value == value and labelOverlay
        table.insert(library.theme, labelOverlay)
        library:AddConnection(label.InputBegan, function(input)
            if input.UserInputType.Name == "MouseButton1" then
                if self.multiselect then
                    self.value[value] = not self.value[value]
                    self:SetValue(self.value)
                else
                    self:SetValue(value)
                    self:Close()
                end
            end
        end)
    end
    for i, value in next, option.values do
        option:AddValue(tostring(typeof(i) == "number" and value or i))
    end
    function option:RemoveValue(value)
        local label = self.labels[value]
        if label then
            label:Destroy()
            self.labels[value] = nil
            valueCount = valueCount - 1
            if self.multiselect then
                self.values[value] = nil
                self:SetValue(self.value)
            else
                table.remove(self.values, table.find(self.values, value))
                if self.value == value then
                    selected = nil
                    self:SetValue(self.values[1] or "")
                end
            end
        end
    end
    function option:SetValue(value, nocallback)
        if self.multiselect and typeof(value) ~= "table" then
            value = {}
            for i, v in next, self.values do
                value[v] = false
            end
        end
        self.value = typeof(value) == "table" and value or tostring(table.find(self.values, value) and value or self.values[1])
        library.flags[self.flag] = self.value
        option.listvalue.Text = " " .. (self.multiselect and getMultiText() or self.value)
        if self.multiselect then
            for name, label in next, self.labels do
                label.TextTransparency = self.value[name] and 1 or 0
                if label:FindFirstChild"TextLabel" then
                    label.TextLabel.Visible = self.value[name]
                end
            end
        else
            if selected then
                selected.TextTransparency = 0
                if selected:FindFirstChild"TextLabel" then
                    selected.TextLabel.Visible = false
                end
            end
            if self.labels[self.value] then
                selected = self.labels[self.value]
                selected.TextTransparency = 1
                if selected:FindFirstChild"TextLabel" then
                    selected.TextLabel.Visible = true
                end
            end
        end
        if not nocallback then
            self.callback(self.value)
        end
    end
    task.delay(1, function()
        if library then
            option:SetValue(option.value)
        end
    end)
    function option:Close()
        library.popup = nil
        option.arrow.Rotation = 90
        self.open = false
        option.holder.Visible = false
        option.listvalue.BorderColor3 = Color3.new()
    end
    return option
end

local function createBox(option, parent)
    option.hasInit = true
    option.main = library:Create("Frame", {
        LayoutOrder = option.position,
        Size = UDim2.new(1, 0, 0, option.text == "nil" and 28 or 44),
        BackgroundTransparency = 1,
        Parent = parent
    })
    if option.text ~= "nil" and not option.sub then
        option.title = library:Create("TextLabel", {
            Position = UDim2.new(0, 6, 0, 0),
            Size = UDim2.new(1, -12, 0, 18),
            BackgroundTransparency = 1,
            Text = option.text,
            TextSize = 15,
            Font = Enum.Font.Code,
            TextColor3 = Color3.fromRGB(210, 210, 210),
            TextXAlignment = Enum.TextXAlignment.Left,
            Parent = option.main
        })
    end
    option.holder = library:Create("Frame", {
        Position = UDim2.new(0, 6, 0, option.text == "nil" and 4 or 20),
        Size = UDim2.new(1, -12, 0, 20),
        BackgroundColor3 = Color3.fromRGB(50, 50, 50),
        BorderColor3 = Color3.new(),
        Parent = option.main
    })
    library:Create("ImageLabel", {
        Size = UDim2.new(1, 0, 1, 0),
        BackgroundTransparency = 1,
        Image = "rbxassetid://2454009026",
        ImageColor3 = Color3.new(),
        ImageTransparency = 0.8,
        Parent = option.holder
    })
    library:Create("ImageLabel", {
        Size = UDim2.new(1, 0, 1, 0),
        BackgroundTransparency = 1,
        Image = "rbxassetid://2592362371",
        ImageColor3 = Color3.fromRGB(60, 60, 60),
        ScaleType = Enum.ScaleType.Slice,
        SliceCenter = Rect.new(2, 2, 62, 62),
        Parent = option.holder
    })
    library:Create("ImageLabel", {
        Size = UDim2.new(1, -2, 1, -2),
        Position = UDim2.new(0, 1, 0, 1),
        BackgroundTransparency = 1,
        Image = "rbxassetid://2592362371",
        ImageColor3 = Color3.new(),
        ScaleType = Enum.ScaleType.Slice,
        SliceCenter = Rect.new(2, 2, 62, 62),
        Parent = option.holder
    })
    local inputvalue = library:Create("TextBox", {
        Position = UDim2.new(0, 4, 0, 1),
        Size = UDim2.new(1, -8, 1, -2),
        BackgroundTransparency = 1,
        Text = option.value,
        TextSize = 15,
        Font = Enum.Font.Code,
        TextColor3 = Color3.new(1, 1, 1),
        TextXAlignment = Enum.TextXAlignment.Left,
        ClearTextOnFocus = false,
        MultiLine = false,
        Parent = option.holder
    })
    local function update()
        local bounds = TextService:GetTextSize(string.sub(inputvalue.Text,1,inputvalue.CursorPosition), 15, Enum.Font.Code, Vector2.new(9e9, 9e9)).X
        local pixels = inputvalue.AbsoluteSize.X - 3
        if bounds >= pixels then
            inputvalue.TextXAlignment = Enum.TextXAlignment.Right
        else
            inputvalue.TextXAlignment = Enum.TextXAlignment.Left
        end
    end
    library:AddConnection(inputvalue:GetPropertyChangedSignal("Text"), function()
        option:SetValue(inputvalue.Text, false)
        update()
    end)
    library:AddConnection(inputvalue:GetPropertyChangedSignal("CursorPosition"), update)
    library:AddConnection(inputvalue:GetPropertyChangedSignal("SelectionStart"), update)
    library:AddConnection(inputvalue.FocusLost, function(enter)
        option.holder.BorderColor3 = Color3.new()
        option:SetValue(inputvalue.Text, enter)
    end)
    library:AddConnection(inputvalue.Focused, function()
        option.holder.BorderColor3 = library.flags["Menu Accent Color"]
    end)
    library:AddConnection(inputvalue.InputBegan, function(input)
        if input.UserInputType.Name == "MouseMovement" then
            if not library.warning and not library.slider then
                option.holder.BorderColor3 = library.flags["Menu Accent Color"]
            end
            if option.tip then
                library.tooltip.Text = option.tip
                local size = TextService:GetTextSize(option.tip, 15, Enum.Font.Code, Vector2.new(9e9, 9e9))
                library.tooltip.Size = UDim2.new(0, size.X, 0, size.Y)
            end
        end
    end)
    library:AddConnection(inputvalue.InputChanged, function(input)
        if input.UserInputType.Name == "MouseMovement" then
            if option.tip then
                library.tooltip.Position = UDim2.new(0, input.Position.X + 26, 0, input.Position.Y + 36)
            end
        end
    end)
    library:AddConnection(inputvalue.InputEnded, function(input)
        if input.UserInputType.Name == "MouseMovement" then
            if not inputvalue:IsFocused() then
                option.holder.BorderColor3 = Color3.new()
            end
            library.tooltip.Position = UDim2.new(2)
        end
    end)
    function option:SetValue(value, enter)
        if not value then
            return
        end
        library.flags[self.flag] = tostring(value)
        self.value = tostring(value)
        inputvalue.Text = self.value
        self.callback(value, enter)
    end
    task.delay(1, function()
        if library then
            option:SetValue(option.value)
        end
    end)
end

local function createColorPickerWindow(option)
    option.mainHolder = library:Create("TextButton", {
        ZIndex = 4,
            --Position = UDim2.new(1, -184, 1, 6),
        Size = UDim2.new(0, option.trans and 200 or 184, 0, 200),
        BackgroundColor3 = Color3.fromRGB(40, 40, 40),
        BorderColor3 = Color3.new(),
        AutoButtonColor = false,
        Visible = false,
        Parent = library.base
    })
    library:Create("ImageLabel", {
        ZIndex = 4,
        Size = UDim2.new(1, 0, 1, 0),
        BackgroundTransparency = 1,
        Image = "rbxassetid://2592362371",
        ImageColor3 = Color3.fromRGB(60, 60, 60),
        ScaleType = Enum.ScaleType.Slice,
        SliceCenter = Rect.new(2, 2, 62, 62),
        Parent = option.mainHolder
    })
    library:Create("ImageLabel", {
        ZIndex = 4,
        Size = UDim2.new(1, -2, 1, -2),
        Position = UDim2.new(0, 1, 0, 1),
        BackgroundTransparency = 1,
        Image = "rbxassetid://2592362371",
        ImageColor3 = Color3.new(),
        ScaleType = Enum.ScaleType.Slice,
        SliceCenter = Rect.new(2, 2, 62, 62),
        Parent = option.mainHolder
    })
    local hue, sat, val = Color3.toHSV(option.color)
    hue, sat, val = hue == 0 and 1 or hue, sat + 0.005, val - 0.005
    local editinghue
    local editingsatval
    local editingtrans
    local transMain
    if option.trans then
        transMain = library:Create("ImageLabel", {
            ZIndex = 5,
            Size = UDim2.new(1, 0, 1, 0),
            BackgroundTransparency = 1,
            Image = "rbxassetid://2454009026",
            ImageColor3 = Color3.fromHSV(hue, 1, 1),
            Rotation = 180,
            Parent = library:Create("ImageLabel", {
                ZIndex = 4,
                AnchorPoint = Vector2.new(1, 0),
                Position = UDim2.new(1, -6, 0, 6),
                Size = UDim2.new(0, 10, 1, -12),
                BorderColor3 = Color3.new(),
                Image = "rbxassetid://4632082392",
                ScaleType = Enum.ScaleType.Tile,
                TileSize = UDim2.new(0, 5, 0, 5),
                Parent = option.mainHolder
            })
        })
        option.transSlider = library:Create("Frame", {
            ZIndex = 5,
            Position = UDim2.new(0, 0, option.trans, 0),
            Size = UDim2.new(1, 0, 0, 2),
            BackgroundColor3 = Color3.fromRGB(38, 41, 65),
            BorderColor3 = Color3.fromRGB(255, 255, 255),
            Parent = transMain
        })
        library:AddConnection(transMain.InputBegan, function(Input)
            if Input.UserInputType.Name == "MouseButton1" then
                editingtrans = true
                option:SetTrans(1 - ((Input.Position.Y - transMain.AbsolutePosition.Y) / transMain.AbsoluteSize.Y))
            end
        end)
        library:AddConnection(transMain.InputEnded, function(Input)
            if Input.UserInputType.Name == "MouseButton1" then
                editingtrans = false
            end
        end)
    end
    local hueMain = library:Create("Frame", {
        ZIndex = 4,
        AnchorPoint = Vector2.new(0, 1),
        Position = UDim2.new(0, 6, 1, -6),
        Size = UDim2.new(1, option.trans and -28 or -12, 0, 10),
        BackgroundColor3 = Color3.new(1, 1, 1),
        BorderColor3 = Color3.new(),
        Parent = option.mainHolder
    })
    local Gradient = library:Create("UIGradient", {
        Color = ColorSequence.new({
            ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 0, 0)),
            ColorSequenceKeypoint.new(0.17, Color3.fromRGB(255, 0, 255)),
            ColorSequenceKeypoint.new(0.33, Color3.fromRGB(0, 0, 255)),
            ColorSequenceKeypoint.new(0.5, Color3.fromRGB(0, 255, 255)),
            ColorSequenceKeypoint.new(0.67, Color3.fromRGB(0, 255, 0)),
            ColorSequenceKeypoint.new(0.83, Color3.fromRGB(255, 255, 0)),
            ColorSequenceKeypoint.new(1, Color3.fromRGB(255, 0, 0)),
        }),
        Parent = hueMain
    })
    local hueSlider = library:Create("Frame", {
        ZIndex = 4,
        Position = UDim2.new(1 - hue, 0, 0, 0),
        Size = UDim2.new(0, 2, 1, 0),
        BackgroundColor3 = Color3.fromRGB(38, 41, 65),
        BorderColor3 = Color3.fromRGB(255, 255, 255),
        Parent = hueMain
    })
    library:AddConnection(hueMain.InputBegan, function(Input)
        if Input.UserInputType.Name == "MouseButton1" then
            editinghue = true
            X = (hueMain.AbsolutePosition.X + hueMain.AbsoluteSize.X) - hueMain.AbsolutePosition.X
            X = math.clamp((Input.Position.X - hueMain.AbsolutePosition.X) / X, 0, 0.995)
            option:SetColor(Color3.fromHSV(1 - X, sat, val))
        end
    end)
    library:AddConnection(hueMain.InputEnded, function(Input)
        if Input.UserInputType.Name == "MouseButton1" then
            editinghue = false
        end
    end)
    local satval = library:Create("ImageLabel", {
        ZIndex = 4,
        Position = UDim2.new(0, 6, 0, 6),
        Size = UDim2.new(1, option.trans and -28 or -12, 1, -28),
        BackgroundColor3 = Color3.fromHSV(hue, 1, 1),
        BorderColor3 = Color3.new(),
        Image = "rbxassetid://4155801252",
        ClipsDescendants = true,
        Parent = option.mainHolder
    })
    local satvalSlider = library:Create("Frame", {
        ZIndex = 4,
        AnchorPoint = Vector2.new(0.5, 0.5),
        Position = UDim2.new(sat, 0, 1 - val, 0),
        Size = UDim2.new(0, 4, 0, 4),
        Rotation = 45,
        BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        Parent = satval
    })
    library:AddConnection(satval.InputBegan, function(Input)
        if Input.UserInputType.Name == "MouseButton1" then
            editingsatval = true
            X = math.clamp((Input.Position.X - satval.AbsolutePosition.X) / satval.AbsoluteSize.X, 0.005, 1)
            Y = math.clamp((Input.Position.Y - satval.AbsolutePosition.Y) / satval.AbsoluteSize.Y, 0, 0.995)
            option:SetColor(Color3.fromHSV(hue, X, 1 - Y))
        end
    end)
    library:AddConnection(RunService.RenderStepped, function()
        local mouse = Players.LocalPlayer:GetMouse()
        if editingsatval then
            X = math.clamp((mouse.X - satval.AbsolutePosition.X) / satval.AbsoluteSize.X, 0.005, 1)
            Y = math.clamp((mouse.Y - satval.AbsolutePosition.Y) / satval.AbsoluteSize.Y, 0, 0.995)
            option:SetColor(Color3.fromHSV(hue, X, 1 - Y))
        elseif editinghue then
            X = (hueMain.AbsolutePosition.X + hueMain.AbsoluteSize.X) - hueMain.AbsolutePosition.X
            X = math.clamp((mouse.X - hueMain.AbsolutePosition.X) / X, 0, 0.995)
            option:SetColor(Color3.fromHSV(1 - X, sat, val))
        elseif editingtrans then
            option:SetTrans(1 - ((mouse.Y - transMain.AbsolutePosition.Y) / transMain.AbsoluteSize.Y))
        end
    end)
    library:AddConnection(satval.InputEnded, function(Input)
        if Input.UserInputType.Name == "MouseButton1" then
            editingsatval = false
        end
    end)
    function option:updateVisuals(Color)
        hue, sat, val = Color3.toHSV(Color)
        hue = hue == 0 and 1 or hue
        satval.BackgroundColor3 = Color3.fromHSV(hue, 1, 1)
        if option.trans then
            transMain.ImageColor3 = Color3.fromHSV(hue, 1, 1)
        end
        hueSlider.Position = UDim2.new(1 - hue, 0, 0, 0)
        satvalSlider.Position = UDim2.new(sat, 0, 1 - val, 0)
    end
    return option
end

local function createColor(option, parent)
    option.hasInit = true
    if option.sub then
        option.main = option:getMain()
    else
        option.main = library:Create("Frame", {
            LayoutOrder = option.position,
            Size = UDim2.new(1, 0, 0, 20),
            BackgroundTransparency = 1,
            Parent = parent
        })
        option.title = library:Create("TextLabel", {
            Position = UDim2.new(0, 6, 0, 0),
            Size = UDim2.new(1, -12, 1, 0),
            BackgroundTransparency = 1,
            Text = option.text,
            TextSize = 15,
            Font = Enum.Font.Code,
            TextColor3 = Color3.fromRGB(210, 210, 210),
            TextXAlignment = Enum.TextXAlignment.Left,
            Parent = option.main
        })
    end
    option.visualize = library:Create(option.sub and "TextButton" or "Frame", {
        Position = UDim2.new(1, -(option.subpos or 0) - 24, 0, 4),
        Size = UDim2.new(0, 18, 0, 12),
        SizeConstraint = Enum.SizeConstraint.RelativeYY,
        BackgroundColor3 = option.color,
        BorderColor3 = Color3.new(),
        Parent = option.main
    })
    library:Create("ImageLabel", {
        Size = UDim2.new(1, 0, 1, 0),
        BackgroundTransparency = 1,
        Image = "rbxassetid://2454009026",
        ImageColor3 = Color3.new(),
        ImageTransparency = 0.6,
        Parent = option.visualize
    })
    library:Create("ImageLabel", {
        Size = UDim2.new(1, 0, 1, 0),
        BackgroundTransparency = 1,
        Image = "rbxassetid://2592362371",
        ImageColor3 = Color3.fromRGB(60, 60, 60),
        ScaleType = Enum.ScaleType.Slice,
        SliceCenter = Rect.new(2, 2, 62, 62),
        Parent = option.visualize
    })
    library:Create("ImageLabel", {
        Size = UDim2.new(1, -2, 1, -2),
        Position = UDim2.new(0, 1, 0, 1),
        BackgroundTransparency = 1,
        Image = "rbxassetid://2592362371",
        ImageColor3 = Color3.new(),
        ScaleType = Enum.ScaleType.Slice,
        SliceCenter = Rect.new(2, 2, 62, 62),
        Parent = option.visualize
    })
    local interest = option.sub and option.visualize or option.main
    if option.sub then
        option.visualize.Text = ""
        option.visualize.AutoButtonColor = false
    end
    library:AddConnection(interest.InputBegan, function(input)
        if input.UserInputType.Name == "MouseButton1" then
            if not option.mainHolder then
                createColorPickerWindow(option)
            end
            if library.popup == option then
                library.popup:Close()
                return
            end
            if library.popup then
                library.popup:Close()
            end
            option.open = true
            local pos = option.main.AbsolutePosition
            option.mainHolder.Position = UDim2.new(0, pos.X + 36 + (option.trans and -16 or 0), 0, pos.Y + 56)
            option.mainHolder.Visible = true
            library.popup = option
            option.visualize.BorderColor3 = library.flags["Menu Accent Color"]
        end
        if input.UserInputType.Name == "MouseMovement" then
            if not library.warning and not library.slider then
                option.visualize.BorderColor3 = library.flags["Menu Accent Color"]
            end
            if option.tip then
                library.tooltip.Text = option.tip
                local size = TextService:GetTextSize(option.tip, 15, Enum.Font.Code, Vector2.new(9e9, 9e9))
                library.tooltip.Size = UDim2.new(0, size.X, 0, size.Y)
            end
        end
    end)
    library:AddConnection(interest.InputChanged, function(input)
        if input.UserInputType.Name == "MouseMovement" then
            if option.tip then
                library.tooltip.Position = UDim2.new(0, input.Position.X + 26, 0, input.Position.Y + 36)
            end
        end
    end)
    library:AddConnection(interest.InputEnded, function(input)
        if input.UserInputType.Name == "MouseMovement" then
            if not option.open then
                option.visualize.BorderColor3 = Color3.new()
            end
            library.tooltip.Position = UDim2.new(2)
        end
    end)
    function option:SetColor(newColor, nocallback)
        if typeof(newColor) == "table" then
            newColor = Color3.new(newColor[1], newColor[2], newColor[3])
        end
        newColor = newColor or Color3.new(1, 1, 1)
        if self.mainHolder then
            self:updateVisuals(newColor)
        end
        option.visualize.BackgroundColor3 = newColor
        library.flags[self.flag] = newColor
        self.color = newColor
        if not nocallback then
            self.callback(newColor)
        end
    end
    if option.trans then
        function option:SetTrans(value, manual)
            value = math.clamp(tonumber(value) or 0, 0, 1)
            if self.transSlider then
                self.transSlider.Position = UDim2.new(0, 0, value, 0)
            end
            self.trans = value
            library.flags[self.flag .. " Transparency"] = value
            self.calltrans(value)
        end
        option:SetTrans(option.trans)
    end
    task.delay(1, function()
        if library then
            option:SetColor(option.color)
        end
    end)
    function option:Close()
        library.popup = nil
        self.open = false
        self.mainHolder.Visible = false
        option.visualize.BorderColor3 = Color3.new()
    end
end
function library:GetTab(title)
    for _, tab in next, self.tabs do
        if tab.title == tostring(title) then
            return tab
        end
    end
end
function library:AddTab(title, pos)
    local tab = {
        canInit = true,
        columns = {},
        title = tostring(title)
    }
    table.insert(self.tabs, pos or #self.tabs + 1, tab)
    function tab:GetColumn(index)
        return self.columns[index] or nil
    end
    function tab:AddPlayerList() 
        local column = {
            position = #self.columns,
            canInit = true,
            isPlrList = true,
            selectedPlayer = nil,
            tab = self
        }
        table.insert(self.columns, column)
        function column:Init()
            if self.hasInit then
                return
            end
            self.hasInit = true
            self.main = library:Create("Frame", {
                Position = UDim2.new(0, 5, 0, 4),
                Size = UDim2.new(1, -10, 0, 350),
                BackgroundColor3 = Color3.fromRGB(30, 30, 30),
                BorderColor3 = Color3.new(),
                Parent = library.columnHolder
            })
            self.content = library:Create("ScrollingFrame", {
                Position = UDim2.new(0, 1, 0, 1), 
                Size = UDim2.new(1, -2, 1, -2),
                BackgroundColor3 = Color3.fromRGB(30, 30, 30),
                BorderColor3 = Color3.new(),
                ScrollBarImageColor3 = Color3.fromRGB(),
                ScrollBarThickness = 0,
                VerticalScrollBarInset = Enum.ScrollBarInset.ScrollBar,
                ScrollingDirection = Enum.ScrollingDirection.Y,
                Parent = library:Create("Frame", {
                    Position = UDim2.new(0, 1, 0, 1),
                    Size = UDim2.new(1, -2, 1, -2),
                    BackgroundTransparency = 0,
                    BorderColor3 = Color3.fromRGB(60, 60, 60),
                    Parent = self.main
                })
            })
            local layout = library:Create("UIListLayout", {
                HorizontalAlignment = Enum.HorizontalAlignment.Center,
                SortOrder = Enum.SortOrder.LayoutOrder,
                Padding = UDim.new(0, 7),
                Parent = self.content
            })
            library:Create("UIPadding", {
                PaddingTop = UDim.new(0, 8),
                PaddingLeft = UDim.new(0, 1),
                PaddingRight = UDim.new(0, 1),
                Parent = self.content
            })
            library:AddConnection(layout.Changed, function()
                self.content.CanvasSize = UDim2.new(0, 0, 0, layout.AbsoluteContentSize.Y + 14)
            end)
            self.main2 = library:Create("Frame", {
                Position = UDim2.new(0, 0, 0, 356),
                Size = UDim2.new(1, 0, 0, 175),
                BackgroundColor3 = Color3.fromRGB(30, 30, 30),
                BorderColor3 = Color3.new(),
                Parent = self.main
            })
            self.content2 = library:Create("Frame", {
                Position = UDim2.new(0, 1, 0, 1), 
                Size = UDim2.new(1, -2, 1, -2),
                BackgroundColor3 = Color3.fromRGB(30, 30, 30),
                BorderColor3 = Color3.new(),
                Parent = library:Create("Frame", {
                    Position = UDim2.new(0, 1, 0, 1),
                    Size = UDim2.new(1, -2, 1, -2),
                    BackgroundTransparency = 0,
                    BorderColor3 = Color3.fromRGB(60, 60, 60),
                    Parent = self.main2
                })
            })
            self.image = library:Create("ImageLabel", {
                Position = UDim2.new(0, 6, 0, 6),
                Size = UDim2.new(0, 100, 0, 100),
                Name = "image",
                BackgroundColor3 = Color3.fromRGB(60, 60, 60),
                BorderColor3 = Color3.new(),
                Parent = self.content2
            })
            self.label = library:Create("TextLabel", {
                Text = "No Player Selected",
                Position = UDim2.new(1, 10, 0, 8),
                TextColor3 = Color3.fromRGB(60, 60, 60),
                TextSize = 18,
                Font = Enum.Font.Code,
                TextXAlignment = Enum.TextXAlignment.Left,
                Parent = self.image
            })  
            self.info = library:Create("TextLabel", {
                Text = "",
                Position = UDim2.new(0, 0, 1, 45),
                TextColor3 = Color3.fromRGB(150, 150, 150),
                TextSize = 13,
                Font = Enum.Font.Code,
                TextXAlignment = Enum.TextXAlignment.Left,
                Parent = self.label
            })  
            self.votekick = library:Create("Frame", {
                Position = UDim2.new(0, 0, 1, -52),
                Size = UDim2.new(0, 117, 0, 26),
                BackgroundTransparency = 1,
                BorderSizePixel = 0,
                Parent = self.content2,
                Visible = false
            })
            self.prioritize = library:Create("Frame", {
                Position = UDim2.new(0, 0, 1, -27),
                Size = UDim2.new(0, 117, 0, 26),
                BackgroundTransparency = 1,
                BorderSizePixel = 0,
                Parent = self.content2,
                Visible = false
            })
            createButton({
                text = "Prioritize",
                flag = "Prioritize",
                canInit = true,
                callback = function() 
                    if self.selectedPlayer then
                        if not library.prioritized[self.selectedPlayer] then
                            library.prioritized[self.selectedPlayer] = false
                        end
                        library.prioritized[self.selectedPlayer] = not library.prioritized[self.selectedPlayer]
                    end
                end
            }, self.prioritize)
            self.ignore = library:Create("Frame", {
                Position = UDim2.new(0, 112, 1, -27),
                Size = UDim2.new(0, 117, 0, 26),
                BackgroundTransparency = 1,
                BorderSizePixel = 0,
                Parent = self.content2,
                Visible = false
            })
            createButton({
                text = "Ignore",
                flag = "Ignore",
                canInit = true,
                callback = function() 
                    if self.selectedPlayer then
                        if not library.ignored[self.selectedPlayer] then
                            library.ignored[self.selectedPlayer] = false
                        end
                        library.ignored[self.selectedPlayer] = not library.ignored[self.selectedPlayer]
                    end
                end
            }, self.ignore)
            self.spectate = library:Create("Frame", {
                Position = UDim2.new(0, 112, 1, -52),
                Size = UDim2.new(0, 117, 0, 26),
                BackgroundTransparency = 1,
                BorderSizePixel = 0,
                Parent = self.content2,
                Visible = false
            })
            self.tpto = library:Create("Frame", {
                Position = UDim2.new(0, 224, 1, -27),
                Size = UDim2.new(0, 117, 0, 26),
                BackgroundTransparency = 1,
                BorderSizePixel = 0,
                Parent = self.content2,
                Visible = false
            })
            self.copyuid = library:Create("Frame", {
                Position = UDim2.new(0, 224, 1, -52),
                Size = UDim2.new(0, 117, 0, 26),
                BackgroundTransparency = 1,
                BorderSizePixel = 0,
                Parent = self.content2,
                Visible = false
            })
            createButton({
                text = "Copy UID",
                flag = "Copy UID",
                canInit = true,
                callback = function() 
                    if self.selectedPlayer then
                        setclipboard(self.selectedPlayer.UserId)
                    end
                end
            }, self.copyuid)
            local playerThumbnails = { bust = {}, head = {} }
            local function getContent(plr)
                if plr then
                    local Main = self.content:FindFirstChild(plr.Name)
                    if Main then
                        return Main, Main.holder.content
                    end
                end
            end
            local function selectPlayer(plr)
                if plr then
                    if self.selectedPlayer then
                        local main, content = getContent(self.selectedPlayer)
                        if main and content then
                            main.BorderColor3 = Color3.new()
                            content.label.TextColor = plr.TeamColor
                        end
                    end
                    local main, content = getContent(plr)
                    if main and content then
                        main.BorderColor3 = library.flags["Menu Accent Color"]
                        content.label.TextColor3 = library.flags["Menu Accent Color"]
                        self.selectedPlayer = plr
                        self.votekick.Visible = true
                        self.prioritize.Visible = true
                        self.ignore.Visible = true
                        self.spectate.Visible = true
                        self.tpto.Visible = true
                        self.copyuid.Visible = true
                        self.image.Image = playerThumbnails.bust[plr]
                        self.label.Text = plr.Name
                        self.label.TextColor = plr.TeamColor
                        local Loaderboard = Players.LocalPlayer.PlayerGui.Leaderboard.Main:FindFirstChild(plr.Team.Name)
                        local Data = Loaderboard and Loaderboard:FindFirstChild("DataFrame") and Loaderboard.DataFrame:FindFirstChild("Data")
                        local Rank = (Data and Data:FindFirstChild(plr.Name) and Data[plr.Name].Rank.Text) or "Unknown"
                        self.info.Text = string.format("Display Name: %s\nRank: %s\nTeam: %s\nAccount Age: %s Days\nUID: %s", plr.DisplayName, Rank, plr.Team.Name, plr.AccountAge, tostring(plr.UserId))
                    end
                else
                    self.selectedPlayer = nil
                    self.image.Image = ""
                    self.info.Text = ""
                    self.label.Text = "No Player Selected"
                    self.label.TextColor3 = Color3.fromRGB(60, 60, 60)
                    self.votekick.Visible = false
                    self.prioritize.Visible = false
                    self.ignore.Visible = false
                    self.spectate.Visible = false
                    self.tpto.Visible = false
                    self.copyuid.Visible = false
                end
            end
            local function addPlayer(plr)
                if plr and plr ~= Players.LocalPlayer then 
                    spawn(function() 
                        repeat
                            playerThumbnails.head[plr] = Players:GetUserThumbnailAsync(plr.UserId, Enum.ThumbnailType.HeadShot, Enum.ThumbnailSize.Size48x48)
                            task.wait()
                        until playerThumbnails.head[plr]
                        repeat
                            playerThumbnails.bust[plr] = Players:GetUserThumbnailAsync(plr.UserId, Enum.ThumbnailType.AvatarBust, Enum.ThumbnailSize.Size100x100)
                            task.wait()
                        until playerThumbnails.bust[plr]
                        local main = library:Create("Frame", {
                            Position = UDim2.new(0, 6, 0, 2),
                            Size = UDim2.new(1, -9, 0, 40),
                            BackgroundColor3 = Color3.fromRGB(30, 30, 30),
                            BorderColor3 = Color3.new(),
                            Name = plr.Name,
                            Parent = self.content
                        })
                        local holder = library:Create("Frame", {
                            Position = UDim2.new(0, 1, 0, 1),
                            Size = UDim2.new(1, -2, 1, -2),
                            BackgroundTransparency = 0,
                            BorderColor3 = Color3.fromRGB(60, 60, 60),
                            Name = "holder",
                            Parent = main
                        })
                        local content = library:Create("Frame", {
                            Position = UDim2.new(0, 1, 0, 1), 
                            Size = UDim2.new(1, -2, 1, -2),
                            BackgroundColor3 = Color3.fromRGB(30, 30, 30),
                            BorderColor3 = Color3.new(),
                            Name = "content",
                            Parent = holder
                        })
                        local image = library:Create("ImageLabel", {
                            Position = UDim2.new(0, 4, 0, 4),
                            Size = UDim2.new(0, 28, 0, 28),
                            Name = "image",
                            BackgroundColor3 = Color3.fromRGB(60, 60, 60),
                            BorderColor3 = Color3.new(),
                            Image = playerThumbnails.head[plr],
                            Parent = content
                        })
                        local label = library:Create("TextLabel", {
                            Text = plr.Name,
                            Position = UDim2.new(0, 32 + 7, 0, 18),
                            TextColor = plr.TeamColor,
                            TextSize = 16,
                            Name = "label",
                            Font = Enum.Font.Code,
                            TextXAlignment = Enum.TextXAlignment.Left,
                            Parent = content
                        })  
                        local prio_igno = library:Create("TextLabel", {
                            Text = "",
                            Position = UDim2.new(1, -5, 0, 18),
                            TextColor3 = library.flags["Menu Accent Color"],
                            TextSize = 13,
                            Name = "label",
                            Font = Enum.Font.Code,
                            TextXAlignment = Enum.TextXAlignment.Right,
                            Parent = content,
                            Visible = true
                        })  
                        if not self.selectedPlayer and plr then
                            selectPlayer(plr)
                        end
                        local connection
                        connection = library:AddConnection(RunService.Heartbeat, function()
                            if plr and library then
                                if self.selectedPlayer ~= plr then
                                    label.TextColor = plr.TeamColor
                                end
                                label.Text = plr.Name
                                prio_igno.Text = string.format("%s %s %s", library.prioritized[plr] and "Prioritized" or "", library.ignored[plr] and "Ignored" or "", library.spectatingPlayer == plr and "Spectating" or "")
                                prio_igno.Visible = prio_igno.Text ~= " "
                                prio_igno.TextColor3 = library.flags["Menu Accent Color"]
                            else
                                connection:Disconnect()
                            end
                        end)
                        library:AddConnection(main.InputBegan, function(input)
                            if input.UserInputType.Name == "MouseButton1" then
                                if not self.selectedPlayer or self.selectedPlayer ~= plr then
                                    selectPlayer(plr)
                                else
                                    selectPlayer(nil)
                                end
                            end
                            if input.UserInputType.Name == "MouseMovement" and self.selectedPlayer ~= plr then
                                main.BorderColor3 = library.flags["Menu Accent Color"]
                            end
                        end)
                        library:AddConnection(main.InputEnded, function(input)
                            if input.UserInputType.Name == "MouseMovement" and self.selectedPlayer ~= plr then
                                main.BorderColor3 = Color3.new()
                            end
                        end)
                    end)
                end
            end
            local function removePlayer(plr)
                local plr = self.content:FindFirstChild(plr.Name)
                if plr then
                    plr:Destroy()
                end
            end
            for _,v in next, Players:GetPlayers() do
                addPlayer(v)
            end
            library:AddConnection(Players.PlayerAdded, addPlayer)
            library:AddConnection(Players.PlayerRemoving, removePlayer)
        end
        if library.hasInit and self.hasInit then
            column:Init()
        end
        return column
    end
    function tab:AddColumn()
        local column = {
            sections = {},
            position = #self.columns,
            canInit = true,
            tab = self
        }
        table.insert(self.columns, column)
        function column:GetSection(title)
            for _, section in next, self.sections do
                if section.title == tostring(title) then
                    return section
                end
            end
        end
        function column:AddSection(title)
            local section = {
                title = tostring(title),
                options = {},
                canInit = true,
                column = self
            }
            table.insert(self.sections, section)
            function section:AddLabel(text)
                local option = {
                    text = text
                }
                option.section = self
                option.type = "label"
                option.position = #self.options
                option.canInit = true
                table.insert(self.options, option)
                if library.hasInit and self.hasInit then
                    createLabel(option, self.content)
                else
                    option.Init = createLabel
                end
                return option
            end
            function section:AddDivider(text)
                local option = {
                    text = text
                }
                option.section = self
                option.type = "divider"
                option.position = #self.options
                option.canInit = true
                table.insert(self.options, option)
                if library.hasInit and self.hasInit then
                    createDivider(option, self.content)
                else
                    option.Init = createDivider
                end
                return option
            end
            function section:AddToggle(option)
                option = typeof(option) == "table" and option or {}
                option.section = self
                option.text = tostring(option.text)
                option.state = typeof(option.state) == "boolean" and option.state or false
                option.callback = typeof(option.callback) == "function" and option.callback or function()
                end
                option.type = "toggle"
                option.position = #self.options
                option.flag = (library.flagprefix and library.flagprefix .. " " or "") .. (option.flag or option.text)
                option.subcount = 0
                option.sidesubcount = 0
                option.canInit = (option.canInit ~= nil and option.canInit) or true
                option.tip = option.tip and tostring(option.tip)
                option.style = option.style == 2
                library.flags[option.flag] = option.state
                table.insert(self.options, option)
                library.options[option.flag] = option
                function option:AddColor(subOption)
                    subOption = typeof(subOption) == "table" and subOption or {}
                    subOption.sub = true
                    subOption.subpos = self.sidesubcount * 24
                    function subOption:getMain()
                        return option.main
                    end
                    self.subcount = self.subcount + 1
                    self.sidesubcount = self.sidesubcount + 1
                    return section:AddColor(subOption)
                end
                function option:AddBind(subOption)
                    subOption = typeof(subOption) == "table" and subOption or {}
                    subOption.sub = true
                    subOption.subpos = self.sidesubcount * 24
                    function subOption:getMain()
                        return option.main
                    end
                    self.subcount = self.subcount + 1
                    self.sidesubcount = self.sidesubcount + 1
                    return section:AddBind(subOption)
                end
                function option:AddList(subOption)
                    subOption = typeof(subOption) == "table" and subOption or {}
                    subOption.sub = true
                    function subOption:getMain()
                        return option.main
                    end
                    self.subcount = self.subcount + 1
                    return section:AddList(subOption)
                end
                function option:AddSlider(subOption)
                    subOption = typeof(subOption) == "table" and subOption or {}
                    subOption.sub = true
                    function subOption:getMain()
                        return option.main
                    end
                    self.subcount = self.subcount + 1
                    return section:AddSlider(subOption)
                end
                function option:AddBox(subOption)
                    subOption = typeof(subOption) == "table" and subOption or {}
                    subOption.sub = true
                    function subOption:getMain()
                        return option.main
                    end
                    self.subcount = self.subcount + 1
                    return section:AddBox(subOption)
                end
                if library.hasInit and self.hasInit then
                    createToggle(option, self.content)
                else
                    option.Init = createToggle
                end
                return option
            end
            function section:AddButton(option)
                option = typeof(option) == "table" and option or {}
                option.section = self
                option.text = tostring(option.text)
                option.callback = typeof(option.callback) == "function" and option.callback or function()
                end
                option.type = "button"
                option.position = #self.options
                option.flag = (library.flagprefix and library.flagprefix .. " " or "") .. (option.flag or option.text)
                option.subcount = 0
                option.canInit = (option.canInit ~= nil and option.canInit) or true
                option.tip = option.tip and tostring(option.tip)
                table.insert(self.options, option)
                library.options[option.flag] = option
                function option:AddBind(subOption)
                    subOption = typeof(subOption) == "table" and subOption or {}
                    subOption.sub = true
                    subOption.subpos = self.subcount * 24
                    function subOption:getMain()
                        option.main.Size = UDim2.new(1, 0, 0, 40)
                        return option.main
                    end
                    self.subcount = self.subcount + 1
                    return section:AddBind(subOption)
                end
                function option:AddColor(subOption)
                    subOption = typeof(subOption) == "table" and subOption or {}
                    subOption.sub = true
                    subOption.subpos = self.subcount * 24
                    function subOption:getMain()
                        option.main.Size = UDim2.new(1, 0, 0, 40)
                        return option.main
                    end
                    self.subcount = self.subcount + 1
                    return section:AddColor(subOption)
                end
                if library.hasInit and self.hasInit then
                    createButton(option, self.content)
                else
                    option.Init = createButton
                end
                return option
            end
            function section:AddBind(option)
                option = typeof(option) == "table" and option or {}
                option.section = self
                option.text = tostring(option.text)
                option.key = (option.key and option.key.Name) or option.key or "none"
                option.nomouse = typeof(option.nomouse) == "boolean" and option.nomouse or false
                option.mode = typeof(option.mode) == "string" and option.mode or "toggle"
                option.callback = typeof(option.callback) == "function" and option.callback or function()
                end
                option.keycallback = typeof(option.callback) == "function" and option.keycallback or function()
                end
                option.type = "bind"
                option.position = #self.options
                option.flag = (library.flagprefix and library.flagprefix .. " " or "") .. (option.flag or option.text)
                option.canInit = (option.canInit ~= nil and option.canInit) or true
                option.tip = option.tip and tostring(option.tip)
                table.insert(self.options, option)
                library.options[option.flag] = option
                if library.hasInit and self.hasInit then
                    createBind(option, self.content)
                else
                    option.Init = createBind
                end
                return option
            end
            function section:AddSlider(option)
                option = typeof(option) == "table" and option or {}
                option.section = self
                option.text = tostring(option.text)
                option.min = typeof(option.min) == "number" and option.min or 0
                option.max = typeof(option.max) == "number" and option.max or 0
                option.value = option.min < 0 and 0 or math.clamp(typeof(option.value) == "number" and option.value or option.min, option.min, option.max)
                option.callback = typeof(option.callback) == "function" and option.callback or function()
                end
                option.float = typeof(option.value) == "number" and option.float or 1
                option.suffix = option.suffix and tostring(option.suffix) or ""
                option.textpos = option.textpos == 2
                option.type = "slider"
                option.position = #self.options
                option.flag = (library.flagprefix and library.flagprefix .. " " or "") .. (option.flag or option.text)
                option.subcount = 0
                option.canInit = (option.canInit ~= nil and option.canInit) or true
                option.tip = option.tip and tostring(option.tip)
                library.flags[option.flag] = option.value
                table.insert(self.options, option)
                library.options[option.flag] = option
                function option:AddColor(subOption)
                    subOption = typeof(subOption) == "table" and subOption or {}
                    subOption.sub = true
                    subOption.subpos = self.subcount * 24
                    function subOption:getMain()
                        return option.main
                    end
                    self.subcount = self.subcount + 1
                    return section:AddColor(subOption)
                end
                function option:AddBind(subOption)
                    subOption = typeof(subOption) == "table" and subOption or {}
                    subOption.sub = true
                    subOption.subpos = self.subcount * 24
                    function subOption:getMain()
                        return option.main
                    end
                    self.subcount = self.subcount + 1
                    return section:AddBind(subOption)
                end
                if library.hasInit and self.hasInit then
                    createSlider(option, self.content)
                else
                    option.Init = createSlider
                end
                return option
            end
            function section:AddList(option)
                option = typeof(option) == "table" and option or {}
                option.section = self
                option.text = tostring(option.text)
                option.values = typeof(option.values) == "table" and option.values or {}
                option.callback = typeof(option.callback) == "function" and option.callback or function()
                end
                option.multiselect = typeof(option.multiselect) == "boolean" and option.multiselect or false
                    --option.groupbox = (not option.multiselect) and (typeof(option.groupbox) == "boolean" and option.groupbox or false)
                option.value = option.multiselect and (typeof(option.value) == "table" and option.value or {}) or tostring(option.value or option.values[1] or "")
                if option.multiselect then
                    for i, v in next, option.values do
                        option.value[v] = false
                    end
                end
                option.max = option.max or 4
                option.open = false
                option.type = "list"
                option.position = #self.options
                option.labels = {}
                option.flag = (library.flagprefix and library.flagprefix .. " " or "") .. (option.flag or option.text)
                option.subcount = 0
                option.canInit = (option.canInit ~= nil and option.canInit) or true
                option.tip = option.tip and tostring(option.tip)
                library.flags[option.flag] = option.value
                table.insert(self.options, option)
                library.options[option.flag] = option
                function option:AddValue(value, state)
                    if self.multiselect then
                        self.values[value] = state
                    else
                        table.insert(self.values, value)
                    end
                end
                function option:AddColor(subOption)
                    subOption = typeof(subOption) == "table" and subOption or {}
                    subOption.sub = true
                    subOption.subpos = self.subcount * 24
                    function subOption:getMain()
                        return option.main
                    end
                    self.subcount = self.subcount + 1
                    return section:AddColor(subOption)
                end
                function option:AddBind(subOption)
                    subOption = typeof(subOption) == "table" and subOption or {}
                    subOption.sub = true
                    subOption.subpos = self.subcount * 24
                    function subOption:getMain()
                        return option.main
                    end
                    self.subcount = self.subcount + 1
                    return section:AddBind(subOption)
                end
                if library.hasInit and self.hasInit then
                    createList(option, self.content)
                else
                    option.Init = createList
                end
                return option
            end
            function section:AddBox(option)
                option = typeof(option) == "table" and option or {}
                option.section = self
                option.text = tostring(option.text)
                option.value = tostring(option.value or "")
                option.callback = typeof(option.callback) == "function" and option.callback or function()
                end
                option.type = "box"
                option.position = #self.options
                option.flag = (library.flagprefix and library.flagprefix .. " " or "") .. (option.flag or option.text)
                option.canInit = (option.canInit ~= nil and option.canInit) or true
                option.tip = option.tip and tostring(option.tip)
                library.flags[option.flag] = option.value
                table.insert(self.options, option)
                library.options[option.flag] = option
                if library.hasInit and self.hasInit then
                    createBox(option, self.content)
                else
                    option.Init = createBox
                end
                return option
            end
            function section:AddColor(option)
                option = typeof(option) == "table" and option or {}
                option.section = self
                option.text = tostring(option.text)
                option.color = typeof(option.color) == "table" and Color3.new(option.color[1], option.color[2], option.color[3]) or option.color or Color3.new(1, 1, 1)
                option.callback = typeof(option.callback) == "function" and option.callback or function()
                end
                option.calltrans = typeof(option.calltrans) == "function" and option.calltrans or (option.calltrans == 1 and option.callback) or function()
                end
                option.open = false
                option.trans = option.trans and tonumber(option.trans)
                option.subcount = 1
                option.type = "color"
                option.position = #self.options
                option.flag = (library.flagprefix and library.flagprefix .. " " or "") .. (option.flag or option.text)
                option.canInit = (option.canInit ~= nil and option.canInit) or true
                option.tip = option.tip and tostring(option.tip)
                library.flags[option.flag] = option.color
                table.insert(self.options, option)
                library.options[option.flag] = option
                function option:AddColor(subOption)
                    subOption = typeof(subOption) == "table" and subOption or {}
                    subOption.sub = true
                    subOption.subpos = self.subcount * 24
                    function subOption:getMain()
                        return option.main
                    end
                    self.subcount = self.subcount + 1
                    return section:AddColor(subOption)
                end
                if option.trans then
                    library.flags[option.flag .. " Transparency"] = option.trans
                end
                if library.hasInit and self.hasInit then
                    createColor(option, self.content)
                else
                    option.Init = createColor
                end
                return option
            end
            function section:SetTitle(newTitle)
                self.title = tostring(newTitle)
                if self.titleText then
                    self.titleText.Text = tostring(newTitle)
                    self.titleText.Size = UDim2.new(0, TextService:GetTextSize(self.title, 15, Enum.Font.Code, Vector2.new(9e9, 9e9)).X + 10, 0, 3)
                end
            end
            function section:Init()
                if self.hasInit then
                    return
                end
                self.hasInit = true
                self.main = library:Create("Frame", {
                    BackgroundColor3 = Color3.fromRGB(30, 30, 30),
                    BorderColor3 = Color3.new(),
                    Parent = column.main
                })
                self.content = library:Create("Frame", {
                    Size = UDim2.new(1, 0, 1, 0),
                    BackgroundColor3 = Color3.fromRGB(30, 30, 30),
                    BorderColor3 = Color3.fromRGB(60, 60, 60),
                    BorderMode = Enum.BorderMode.Inset,
                    Parent = self.main
                })
                library:Create("ImageLabel", {
                    Size = UDim2.new(1, -2, 1, -2),
                    Position = UDim2.new(0, 1, 0, 1),
                    BackgroundTransparency = 1,
                    Image = "rbxassetid://2592362371",
                    ImageColor3 = Color3.new(),
                    ScaleType = Enum.ScaleType.Slice,
                    SliceCenter = Rect.new(2, 2, 62, 62),
                    Parent = self.main
                })
                table.insert(library.theme, library:Create("Frame", {
                    Size = UDim2.new(1, 0, 0, 1),
                    BackgroundColor3 = library.flags["Menu Accent Color"],
                    BorderSizePixel = 0,
                    BorderMode = Enum.BorderMode.Inset,
                    Parent = self.main
                }))
                local layout = library:Create("UIListLayout", {
                    HorizontalAlignment = Enum.HorizontalAlignment.Center,
                    SortOrder = Enum.SortOrder.LayoutOrder,
                    Padding = UDim.new(0, 2),
                    Parent = self.content
                })
                library:Create("UIPadding", {
                    PaddingTop = UDim.new(0, 12),
                    Parent = self.content
                })
                self.titleText = library:Create("TextLabel", {
                    AnchorPoint = Vector2.new(0, 0.5),
                    Position = UDim2.new(0, 12, 0, 0),
                    Size = UDim2.new(0, TextService:GetTextSize(self.title, 15, Enum.Font.Code, Vector2.new(9e9, 9e9)).X + 10, 0, 3),
                    BackgroundColor3 = Color3.fromRGB(30, 30, 30),
                    BorderSizePixel = 0,
                    Text = self.title,
                    TextSize = 15,
                    Font = Enum.Font.Code,
                    TextColor3 = Color3.new(1, 1, 1),
                    Parent = self.main
                })
                library:AddConnection(layout.Changed, function()
                    self.main.Size = UDim2.new(1, 0, 0, layout.AbsoluteContentSize.Y + 16)
                end)
                for _, option in next, self.options do
                    if option.canInit then
                        option.Init(option, self.content)
                    end
                end
            end
            if library.hasInit and self.hasInit then
                section:Init()
            end
            return section
        end
        function column:Init()
            if self.hasInit then
                return
            end
            self.hasInit = true
            self.main = library:Create("ScrollingFrame", {
                ZIndex = 2,
                Position = UDim2.new(0, 6 + (self.position * 239), 0, 2),
                Size = UDim2.new(0, 233, 1, -4),
                BackgroundTransparency = 1,
                BorderSizePixel = 0,
                ScrollBarImageColor3 = Color3.fromRGB(),
                ScrollBarThickness = 4,
                VerticalScrollBarInset = Enum.ScrollBarInset.ScrollBar,
                ScrollingDirection = Enum.ScrollingDirection.Y,
                Visible = false,
                Parent = library.columnHolder
            })
            local layout = library:Create("UIListLayout", {
                HorizontalAlignment = Enum.HorizontalAlignment.Center,
                SortOrder = Enum.SortOrder.LayoutOrder,
                Padding = UDim.new(0, 12),
                Parent = self.main
            })
            library:Create("UIPadding", {
                PaddingTop = UDim.new(0, 8),
                PaddingLeft = UDim.new(0, 2),
                PaddingRight = UDim.new(0, 2),
                Parent = self.main
            })
            library:AddConnection(layout.Changed, function()
                self.main.CanvasSize = UDim2.new(0, 0, 0, layout.AbsoluteContentSize.Y + 14)
            end)
            if not self.isPlrList then
                for _, section in next, self.sections do
                    if section.canInit then
                        if section.options and #section.options > 0 then
                            section:Init()
                        end
                    end
                end
            end
        end
        if library.hasInit and self.hasInit then
            column:Init()
        end
        return column
    end
    function tab:Init()
        if self.hasInit then
            return
        end
        self.hasInit = true
        local size = TextService:GetTextSize(self.title, 18, Enum.Font.Code, Vector2.new(9e9, 9e9)).X + 10
        self.button = library:Create("TextLabel", {
            Position = UDim2.new(0, library.tabSize, 0, 22),
            Size = UDim2.new(0, size, 0, 30),
            BackgroundTransparency = 1,
            Text = self.title,
            TextColor3 = Color3.new(1, 1, 1),
            TextSize = 15,
            Font = Enum.Font.Code,
            TextWrapped = true,
            ClipsDescendants = true,
            Parent = library.main
        })
        library.tabSize = library.tabSize + size
        library:AddConnection(self.button.InputBegan, function(input)
            if input.UserInputType.Name == "MouseButton1" then
                library:selectTab(self)
            end
        end)
        for _, column in next, self.columns do
            if column.canInit then
                column:Init()
            end
        end
    end
    if self.hasInit then
        tab:Init()
    end
    return tab
end
function library:AddWarning(warning)
    warning = typeof(warning) == "table" and warning or {}
    warning.text = tostring(warning.text)
    warning.type = warning.type == "confirm" and "confirm" or ""
    local answer
    function warning:Show()
        library.warning = warning
        if warning.main and warning.type == "" then
            return
        end
        if library.popup then
            library.popup:Close()
        end
        if not warning.main then
            warning.main = library:Create("TextButton", {
                ZIndex = 2,
                Size = UDim2.new(1, 0, 1, 0),
                BackgroundTransparency = 0.6,
                BackgroundColor3 = Color3.new(),
                BorderSizePixel = 0,
                Text = "",
                AutoButtonColor = false,
                Parent = library.main
            })
            warning.message = library:Create("TextLabel", {
                ZIndex = 2,
                Position = UDim2.new(0, 20, 0.5, -60),
                Size = UDim2.new(1, -40, 0, 40),
                BackgroundTransparency = 1,
                TextSize = 16,
                Font = Enum.Font.Code,
                TextColor3 = Color3.new(1, 1, 1),
                TextWrapped = true,
                RichText = true,
                Parent = warning.main
            })
            if warning.type == "confirm" then
                local button = library:Create("TextLabel", {
                    ZIndex = 2,
                    Position = UDim2.new(0.5, -105, 0.5, -10),
                    Size = UDim2.new(0, 100, 0, 20),
                    BackgroundColor3 = Color3.fromRGB(40, 40, 40),
                    BorderColor3 = Color3.new(),
                    Text = "Yes",
                    TextSize = 16,
                    Font = Enum.Font.Code,
                    TextColor3 = Color3.new(1, 1, 1),
                    Parent = warning.main
                })
                library:Create("ImageLabel", {
                    ZIndex = 2,
                    Size = UDim2.new(1, 0, 1, 0),
                    BackgroundTransparency = 1,
                    Image = "rbxassetid://2454009026",
                    ImageColor3 = Color3.new(),
                    ImageTransparency = 0.8,
                    Parent = button
                })
                library:Create("ImageLabel", {
                    ZIndex = 2,
                    Size = UDim2.new(1, 0, 1, 0),
                    BackgroundTransparency = 1,
                    Image = "rbxassetid://2592362371",
                    ImageColor3 = Color3.fromRGB(60, 60, 60),
                    ScaleType = Enum.ScaleType.Slice,
                    SliceCenter = Rect.new(2, 2, 62, 62),
                    Parent = button
                })
                local button1 = library:Create("TextLabel", {
                    ZIndex = 2,
                    Position = UDim2.new(0.5, 5, 0.5, -10),
                    Size = UDim2.new(0, 100, 0, 20),
                    BackgroundColor3 = Color3.fromRGB(40, 40, 40),
                    BorderColor3 = Color3.new(),
                    Text = "No",
                    TextSize = 16,
                    Font = Enum.Font.Code,
                    TextColor3 = Color3.new(1, 1, 1),
                    Parent = warning.main
                })
                library:Create("ImageLabel", {
                    ZIndex = 2,
                    Size = UDim2.new(1, 0, 1, 0),
                    BackgroundTransparency = 1,
                    Image = "rbxassetid://2454009026",
                    ImageColor3 = Color3.new(),
                    ImageTransparency = 0.8,
                    Parent = button1
                })
                library:Create("ImageLabel", {
                    ZIndex = 2,
                    Size = UDim2.new(1, 0, 1, 0),
                    BackgroundTransparency = 1,
                    Image = "rbxassetid://2592362371",
                    ImageColor3 = Color3.fromRGB(60, 60, 60),
                    ScaleType = Enum.ScaleType.Slice,
                    SliceCenter = Rect.new(2, 2, 62, 62),
                    Parent = button1
                })
                library:AddConnection(button.InputBegan, function(input)
                    if input.UserInputType.Name == "MouseButton1" then
                        answer = true
                    end
                end)
                library:AddConnection(button1.InputBegan, function(input)
                    if input.UserInputType.Name == "MouseButton1" then
                        answer = false
                    end
                end)
            else
                local button = library:Create("TextLabel", {
                    ZIndex = 2,
                    Position = UDim2.new(0.5, -50, 0.5, -10),
                    Size = UDim2.new(0, 100, 0, 20),
                    BackgroundColor3 = Color3.fromRGB(30, 30, 30),
                    BorderColor3 = Color3.new(),
                    Text = "OK",
                    TextSize = 16,
                    Font = Enum.Font.Code,
                    TextColor3 = Color3.new(1, 1, 1),
                    Parent = warning.main
                })
                library:Create("ImageLabel", {
                    ZIndex = 2,
                    Size = UDim2.new(1, 0, 1, 0),
                    BackgroundTransparency = 1,
                    Image = "rbxassetid://2454009026",
                    ImageColor3 = Color3.new(),
                    ImageTransparency = 0.8,
                    Parent = button
                })
                library:Create("ImageLabel", {
                    ZIndex = 2,
                    Size = UDim2.new(1, 0, 1, 0),
                    BackgroundTransparency = 1,
                    Image = "rbxassetid://2592362371",
                    ImageColor3 = Color3.fromRGB(60, 60, 60),
                    ScaleType = Enum.ScaleType.Slice,
                    SliceCenter = Rect.new(2, 2, 62, 62),
                    Parent = button
                })
                library:AddConnection(button.InputBegan, function(input)
                    if input.UserInputType.Name == "MouseButton1" then
                        answer = true
                    end
                end)
            end
        end
        warning.main.Visible = true
        warning.message.Text = warning.text
        repeat
            task.wait()
        until answer ~= nil
        spawn(warning.Close)
        library.warning = nil
        return answer
    end
    function warning:Close()
        answer = nil
        if not warning.main then
            return
        end
        warning.main.Visible = false
    end
    return warning
end
function library:Close()
    self.open = not self.open
    if self.main then
        if self.popup then
            self.popup:Close()
        end
        self.main.Visible = self.open
    end
end
function library:Init()
    if self.hasInit then
        return
    end
    self.hasInit = true
    self.base = library:Create("ScreenGui", {
        IgnoreGuiInset = true
    })
    if gethui then
        self.base.Parent = gethui()
    elseif syn and syn.protect_gui then
        syn.protect_gui(self.base)
        self.base.Parent = CoreGui
    end
    self.main = self:Create("ImageButton", {
        AutoButtonColor = false,
        Image = "rbxassetid://5553946656",
        ImageColor3 = Color3.new(),
        TileSize = UDim2.new(0, 90, 0, 90),
        AnchorPoint = Vector2.new(0.5, 0.5),
        Position = UDim2.new(0.5, 0, 0.5, 0),
        Size = UDim2.new(0, 90, 0, 90), --500, 600
        BackgroundColor3 = Color3.fromRGB(20, 20, 20),
        BorderColor3 = Color3.new(),
        ScaleType = Enum.ScaleType.Tile,
        Visible = false,
        Parent = self.base
    })
    self.mainTop = self:Create("Frame", {
        Size = UDim2.new(1, 0, 0, 50),
        BackgroundColor3 = Color3.fromRGB(30, 30, 30),
        BorderColor3 = Color3.new(),
        Parent = self.main
    })
    self.mainGradient = self:Create("UIGradient", {
        Color = ColorSequence.new({
            ColorSequenceKeypoint.new(0, Color3.fromRGB(30, 30, 30)),
            ColorSequenceKeypoint.new(1, Color3.fromRGB(10, 10, 10)),
        }),
        Rotation = 90,
        Parent = self.mainTop,
        Enabled = false
    })
    self.titleLabel = self:Create("TextLabel", {
        Position = UDim2.new(0, 6, 0, -1),
        Size = UDim2.new(0, 0, 0, 20),
        BackgroundTransparency = 1,
        Text = tostring(self.title),
        Font = Enum.Font.Code,
        TextSize = 18,
        TextColor3 = Color3.new(1, 1, 1),
        TextXAlignment = Enum.TextXAlignment.Left,
        Parent = self.main
    })
    table.insert(library.theme, self:Create("Frame", {
        Size = UDim2.new(1, 0, 0, 1),
        Position = UDim2.new(0, 0, 0, 24),
        BackgroundColor3 = library.flags["Menu Accent Color"],
        BorderSizePixel = 0,
        Parent = self.main
    }))
    library:Create("ImageLabel", {
        Size = UDim2.new(1, 0, 1, 0),
        BackgroundTransparency = 1,
        ImageColor3 = Color3.new(),
        ImageTransparency = 0.4,
        Parent = self.mainTop
    })
    self.tabHighlight = self:Create("Frame", {
        BackgroundColor3 = library.flags["Menu Accent Color"],
        BorderSizePixel = 0,
        Parent = self.main
    })
    table.insert(library.theme, self.tabHighlight)
    self.columnHolder = self:Create("Frame", {
        Position = UDim2.new(0, 5, 0, 55),
        Size = UDim2.new(1, -10, 1, -60),
        BackgroundTransparency = 1,
        Parent = self.main
    })
    self.tooltip = self:Create("TextLabel", {
        ZIndex = 2,
        BackgroundTransparency = 1,
        BorderSizePixel = 0,
        TextSize = 15,
        Font = Enum.Font.Code,
        TextColor3 = Color3.new(1, 1, 1),
        Visible = true,
        Parent = self.base
    })
    self:Create("Frame", {
        AnchorPoint = Vector2.new(0.5, 0),
        Position = UDim2.new(0.5, 0, 0, 0),
        Size = UDim2.new(1, 10, 1, 0),
        Style = Enum.FrameStyle.RobloxRound,
        Parent = self.tooltip
    })
    self:Create("ImageLabel", {
        Size = UDim2.new(1, 0, 1, 0),
        BackgroundTransparency = 1,
        Image = "rbxassetid://2592362371",
        ImageColor3 = Color3.fromRGB(60, 60, 60),
        ScaleType = Enum.ScaleType.Slice,
        SliceCenter = Rect.new(2, 2, 62, 62),
        Parent = self.main
    })
    self:Create("ImageLabel", {
        Size = UDim2.new(1, -2, 1, -2),
        Position = UDim2.new(0, 1, 0, 1),
        BackgroundTransparency = 1,
        Image = "rbxassetid://2592362371",
        ImageColor3 = Color3.new(),
        ScaleType = Enum.ScaleType.Slice,
        SliceCenter = Rect.new(2, 2, 62, 62),
        Parent = self.main
    })
    library:AddConnection(self.mainTop.InputBegan, function(input, processed)
        if not processed and input.UserInputType.Name == "MouseButton1" then
            dragging = true
            dragOffset = self.udim2ToVector2(self.main.Position) - UserInputService:GetMouseLocation()
            if library.popup then
                library.popup:Close()
            end
        end
    end)
    library:AddConnection(self.mainTop.InputEnded, function(input)
        if input.UserInputType.Name == "MouseButton1" then
            dragging = false
            dragOffset = nil
        end
    end)
    library:AddConnection(RunService.RenderStepped, function()
        if dragging and dragOffset then
            self.main.Position = self.vector2ToUDim2(UserInputService:GetMouseLocation() + dragOffset)
        end
    end)
    function self:selectTab(tab)
        if self.currentTab == tab then
            return
        end
        if library.popup then
            library.popup:Close()
        end
        if self.currentTab then
            self.currentTab.button.TextColor3 = Color3.fromRGB(255, 255, 255)
            for _, column in next, self.currentTab.columns do
                column.main.Visible = false
            end
        end
        self.main.Size = UDim2.new(0, 16 + ((#tab.columns < 2 and 2 or #tab.columns) * 239), 0, 600)
        self.currentTab = tab
        tab.button.TextColor3 = library.flags["Menu Accent Color"]
        TweenService:Create(self.tabHighlight, TweenInfo.new(0.2), { Position = UDim2.fromOffset(tab.button.Position.X.Offset, 50), Size = UDim2.fromOffset(tab.button.AbsoluteSize.X, -1) }):Play()
        for _, column in next, tab.columns do
            column.main.Visible = true
        end
    end
    spawn(function()
        while library do
            task.wait(1)
            if self.options["Config List"] then
                local Configs = self:GetConfigs()
                for _, config in next, Configs do
                    if not table.find(self.options["Config List"].values, config) then
                        self.options["Config List"]:AddValue(config)
                    end
                end
                for i, config in next, self.options["Config List"].values do
                    if not table.find(Configs, config) then
                        self.options["Config List"]:RemoveValue(config)
                    end
                end
            end
            if self.options["Background List"] then
                local Configs = self:GetBackgrounds()
                for _, config in next, Configs do
                    if not table.find(self.options["Background List"].values, config) then
                        self.options["Background List"]:AddValue(config)
                    end
                end
                for i, config in next, self.options["Background List"].values do
                    if not table.find(Configs, config) then
                        self.options["Background List"]:RemoveValue(config)
                    end
                end
            end
        end
    end)
    for _, tab in next, self.tabs do
        if tab.canInit then
            tab:Init()
            self:selectTab(tab)
        end
    end
    self:Close()
end
return library
