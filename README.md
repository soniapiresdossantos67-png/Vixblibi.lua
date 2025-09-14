-- Vixblibi.lua (versão com layout lateral, key system, tema configurável e animações)
-- Pode ser salvo como arquivo .lua e usado via loadstring(game:HttpGet(...))

local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")

local Vix = {}
Vix._windows = {}

local function make(class, props)
    local obj = Instance.new(class)
    for k,v in pairs(props or {}) do
        obj[k] = v
    end
    return obj
end

local function makeDraggable(frame, dragHandle)
    local dragging, dragStart, startPos
    dragHandle.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStart = input.Position
            startPos = frame.Position
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
            local delta = input.Position - dragStart
            frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)
end

function Vix:CreateWindow(opts)
    opts = opts or {}
    local title = opts.Title or "Vixblibi"
    local size = opts.Size or UDim2.new(0, 600, 0, 400)
    local theme = opts.Theme or {
        Background = Color3.fromRGB(30,30,40),
        Sidebar = Color3.fromRGB(20,20,30),
        Accent = Color3.fromRGB(0,170,140),
        Text = Color3.fromRGB(255,255,255)
    }
    local rounded = opts.Rounded ~= false

    -- Key System
    if opts.KeySystem and opts.KeySystem.Enabled then
        local keyGui = make("ScreenGui", {Parent = game.CoreGui, Name = "VixKey"})
        local bg = make("Frame", {Parent = keyGui, Size = UDim2.new(0,300,0,150), Position = UDim2.new(0.5,-150,0.5,-75), BackgroundColor3 = theme.Background})
        if rounded then make("UICorner", {Parent = bg, CornerRadius = UDim.new(0,8)}) end
        local box = make("TextBox", {Parent = bg, Size = UDim2.new(1,-20,0,30), Position = UDim2.new(0,10,0.5,-15), Text = "", PlaceholderText = "Enter Key", TextColor3 = theme.Text, BackgroundColor3 = theme.Sidebar})
        if rounded then make("UICorner", {Parent = box, CornerRadius = UDim.new(0,6)}) end
        local confirmed = false
        box.FocusLost:Wait()
        if box.Text ~= opts.KeySystem.Key then
            keyGui:Destroy()
            return nil
        else
            confirmed = true
            keyGui:Destroy()
        end
        if not confirmed then return end
    end

    -- GUI base
    local gui = make("ScreenGui", {Parent = game.CoreGui, Name = "Vixblibi"})
    local main = make("Frame", {Parent = gui, Size = size, Position = UDim2.new(0.5,-size.X.Offset/2,0.5,-size.Y.Offset/2), BackgroundColor3 = theme.Background})
    if rounded then make("UICorner", {Parent = main, CornerRadius = UDim.new(0,8)}) end

    local titleBar = make("Frame", {Parent = main, Size = UDim2.new(1,0,0,28), BackgroundColor3 = theme.Sidebar})
    local titleLabel = make("TextLabel", {Parent = titleBar, Size = UDim2.new(1,0,1,0), BackgroundTransparency = 1, Text = title, TextColor3 = theme.Text, Font = Enum.Font.GothamBold, TextSize = 14})

    makeDraggable(main, titleBar)

    local sidebar = make("Frame", {Parent = main, Size = UDim2.new(0,120,1,-28), Position = UDim2.new(0,0,0,28), BackgroundColor3 = theme.Sidebar})
    if rounded then make("UICorner", {Parent = sidebar, CornerRadius = UDim.new(0,6)}) end

    local content = make("Frame", {Parent = main, Size = UDim2.new(1,-120,1,-28), Position = UDim2.new(0,120,0,28), BackgroundColor3 = theme.Background})
    if rounded then make("UICorner", {Parent = content, CornerRadius = UDim.new(0,6)}) end

    local layout = make("UIListLayout", {Parent = sidebar, SortOrder = Enum.SortOrder.LayoutOrder, Padding = UDim.new(0,4)})

    local window = {
        Gui = gui,
        Main = main,
        Sidebar = sidebar,
        Content = content,
        Tabs = {}
    }

    function window:CreateTab(tabName)
        local btn = make("TextButton", {Parent = sidebar, Size = UDim2.new(1,-10,0,28), Text = tabName, BackgroundColor3 = theme.Background, TextColor3 = theme.Text, Font = Enum.Font.Gotham, TextSize = 14})
        if rounded then make("UICorner", {Parent = btn, CornerRadius = UDim.new(0,6)}) end

        local tabFrame = make("ScrollingFrame", {Parent = content, Size = UDim2.new(1,0,1,0), CanvasSize = UDim2.new(0,0), ScrollBarThickness = 6, BackgroundTransparency = 1, Visible = false})
        make("UIListLayout", {Parent = tabFrame, SortOrder = Enum.SortOrder.LayoutOrder, Padding = UDim.new(0,6)})

        btn.MouseButton1Click:Connect(function()
            for _,tab in pairs(self.Tabs) do
                tab.Frame.Visible = false
                TweenService:Create(tab.Btn, TweenInfo.new(0.2), {BackgroundColor3 = theme.Background}):Play()
            end
            tabFrame.Visible = true
            TweenService:Create(btn, TweenInfo.new(0.2), {BackgroundColor3 = theme.Accent}):Play()
        end)

        self.Tabs[tabName] = {Btn = btn, Frame = tabFrame}

        local tab = {}
        function tab:CreateSection(sectionName)
            local sec = make("Frame", {Parent = tabFrame, Size = UDim2.new(1,0,0,120), BackgroundColor3 = theme.Sidebar})
            if rounded then make("UICorner", {Parent = sec, CornerRadius = UDim.new(0,6)}) end
            make("TextLabel", {Parent = sec, Size = UDim2.new(1,0,0,24), Text = sectionName, BackgroundTransparency = 1, TextColor3 = theme.Text, Font = Enum.Font.GothamBold, TextSize = 14})

            local contentHolder = make("Frame", {Parent = sec, Size = UDim2.new(1,0,1,-24), Position = UDim2.new(0,0,0,24), BackgroundTransparency = 1})
            make("UIListLayout", {Parent = contentHolder, Padding = UDim.new(0,4)})

            local api = {}
            function api:Button(text, callback)
                local b = make("TextButton", {Parent = contentHolder, Size = UDim2.new(1,-10,0,28), Text = text, BackgroundColor3 = theme.Background, TextColor3 = theme.Text, Font = Enum.Font.Gotham, TextSize = 14})
                if rounded then make("UICorner", {Parent = b, CornerRadius = UDim.new(0,6)}) end
                b.MouseButton1Click:Connect(function() pcall(callback) end)
            end
            return api
        end
        return tab
    end

    return window
end

return Vix
