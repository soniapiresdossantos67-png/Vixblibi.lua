-- Vixblibi.lua (Atualizada)
-- Versão melhorada: tamanho padrão reduzido, layout com separação entre sidebar e conteúdo,
-- botões minimizar/maximizar/fechar, botão flutuante do jogador para abrir/fechar, e API de comandos.
-- Salve como Vixblibi.lua e use via loadstring(game:HttpGet(...))() ou require em ModuleScript.

local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local Vix = {}
Vix._windows = {}

local function protectGui(gui)
    if syn and syn.protect_gui then
        pcall(function() syn.protect_gui(gui) end)
    end
end

local function new(class, props)
    local obj = Instance.new(class)
    for k,v in pairs(props or {}) do
        if k ~= "Parent" then
            pcall(function() obj[k] = v end)
        end
    end
    if props and props.Parent then obj.Parent = props.Parent end
    return obj
end

local function makeDraggable(frame, handle)
    local dragging, dragStart, startPos
    handle.InputBegan:Connect(function(input)
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

-- Default settings
local DEFAULTS = {
    Size = UDim2.new(0,420,0,320), -- smaller default size
    SidebarWidth = 110,
    Spacing = 12,
    Rounded = true,
    Theme = {
        Background = Color3.fromRGB(28,28,34),
        Sidebar = Color3.fromRGB(22,22,28),
        Divider = Color3.fromRGB(16,16,20),
        Accent = Color3.fromRGB(0,170,140),
        Text = Color3.fromRGB(230,230,230)
    }
}

-- Util: create toggle button visible on player's screen to open/close window
local function createToggleButton(guiParent, theme)
    local btn = new("ImageButton", {
        Parent = guiParent,
        Size = UDim2.new(0,48,0,48),
        Position = UDim2.new(1,-72,1,-88),
        AnchorPoint = Vector2.new(0,0),
        BackgroundColor3 = theme.Sidebar,
        AutoButtonColor = true,
        Image = "",
    })
    new("UICorner", {Parent = btn, CornerRadius = UDim.new(0,12)})
    local ic = new("TextLabel", {Parent = btn, Size = UDim2.new(1,1,0,0), BackgroundTransparency = 1, Text = "≡", TextSize = 28, Font = Enum.Font.GothamBold, TextColor3 = theme.Text})
    return btn
end

-- Window creation
function Vix:CreateWindow(opts)
    opts = opts or {}
    local title = opts.Title or "Vixblibi"
    local size = opts.Size or DEFAULTS.Size
    local sidebarWidth = opts.SidebarWidth or DEFAULTS.SidebarWidth
    local theme = opts.Theme or DEFAULTS.Theme
    local rounded = (opts.Rounded == nil) and DEFAULTS.Rounded or opts.Rounded
    local spacing = opts.Spacing or DEFAULTS.Spacing

    -- Key system (simple) - blocking prompt
    if opts.KeySystem and opts.KeySystem.Enabled then
        local keyGui = new("ScreenGui", {Name = "VixKeyGui"})
        protectGui(keyGui)
        keyGui.Parent = Players.LocalPlayer:WaitForChild("PlayerGui") or game:GetService("CoreGui")

        local keyBg = new("Frame", {Parent = keyGui, Size = UDim2.new(0,320,0,150), Position = UDim2.new(0.5,-160,0.5,-75), BackgroundColor3 = theme.Background})
        if rounded then new("UICorner", {Parent = keyBg, CornerRadius = UDim.new(0,10)}) end
        local prompt = new("TextLabel", {Parent = keyBg, Size = UDim2.new(1,-20,0,36), Position = UDim2.new(0,10,0,10), BackgroundTransparency = 1, Text = opts.KeySystem.Prompt or "Enter Key:", TextColor3 = theme.Text, Font = Enum.Font.GothamBold, TextSize = 16, TextXAlignment = Enum.TextXAlignment.Left})
        local box = new("TextBox", {Parent = keyBg, Size = UDim2.new(1,-20,0,30), Position = UDim2.new(0,10,0,56), BackgroundColor3 = theme.Sidebar, TextColor3 = theme.Text, PlaceholderText = "Key..."})
        if rounded then new("UICorner", {Parent = box, CornerRadius = UDim.new(0,6)}) end
        local okBtn = new("TextButton", {Parent = keyBg, Size = UDim2.new(0,100,0,28), Position = UDim2.new(1,-110,1,-38), Text = "OK", BackgroundColor3 = theme.Accent, TextColor3 = Color3.new(1,1,1)})
        if rounded then new("UICorner", {Parent = okBtn, CornerRadius = UDim.new(0,6)}) end

        local valid = false
        okBtn.MouseButton1Click:Connect(function()
            if tostring(box.Text) == tostring(opts.KeySystem.Key) then
                valid = true
                keyGui:Destroy()
            else
                box.Text = ""
                box.PlaceholderText = "Key inválida"
            end
        end)

        -- block until closed or matched
        local start = tick()
        while keyGui.Parent and tick() - start < 300 do
            RunService.Heartbeat:Wait()
        end
        if keyGui.Parent then keyGui:Destroy() end
        if not valid then return nil end
    end

    -- Base GUI
    local screen = new("ScreenGui", {Name = "VixblibiGui"})
    protectGui(screen)
    screen.Parent = Players.LocalPlayer:FindFirstChild("PlayerGui") or game:GetService("CoreGui")

    -- Main frame
    local main = new("Frame", {Parent = screen, Size = size, Position = UDim2.new(0.5, -size.X.Offset/2, 0.5, -size.Y.Offset/2), BackgroundColor3 = theme.Background})
    if rounded then new("UICorner", {Parent = main, CornerRadius = UDim.new(0,10)}) end
    new("UIStroke", {Parent = main, Color = Color3.fromRGB(0,0,0), Transparency = 0.6, Thickness = 1})

    -- Title bar (for dragging) + control buttons
    local titleBar = new("Frame", {Parent = main, Size = UDim2.new(1,0,0,34), BackgroundColor3 = theme.Sidebar})
    if rounded then new("UICorner", {Parent = titleBar, CornerRadius = UDim.new(0,8)}) end
    local titleLabel = new("TextLabel", {Parent = titleBar, Size = UDim2.new(1,-120,1,0), Position = UDim2.new(0,12,0,0), BackgroundTransparency = 1, Text = title, TextColor3 = theme.Text, Font = Enum.Font.GothamBold, TextSize = 16, TextXAlignment = Enum.TextXAlignment.Left})

    -- Control buttons: minimize, maximize, close
    local btnClose = new("TextButton", {Parent = titleBar, Size = UDim2.new(0,32,0,24), Position = UDim2.new(1,-42,0,6), BackgroundColor3 = theme.Sidebar, Text = "✕", TextColor3 = theme.Text, Font = Enum.Font.GothamSemibold, TextSize = 14})
    local btnMax = new("TextButton", {Parent = titleBar, Size = UDim2.new(0,32,0,24), Position = UDim2.new(1,-82,0,6), BackgroundColor3 = theme.Sidebar, Text = "▢", TextColor3 = theme.Text, Font = Enum.Font.GothamSemibold, TextSize = 12})
    local btnMin = new("TextButton", {Parent = titleBar, Size = UDim2.new(0,32,0,24), Position = UDim2.new(1,-122,0,6), BackgroundColor3 = theme.Sidebar, Text = "—", TextColor3 = theme.Text, Font = Enum.Font.GothamSemibold, TextSize = 18})
    if rounded then new("UICorner", {Parent = btnClose, CornerRadius = UDim.new(0,6)}) end
    if rounded then new("UICorner", {Parent = btnMax, CornerRadius = UDim.new(0,6)}) end
    if rounded then new("UICorner", {Parent = btnMin, CornerRadius = UDim.new(0,6)}) end

    -- Sidebar and content with divider
    local sidebar = new("Frame", {Parent = main, Size = UDim2.new(0,sidebarWidth,1,-34), Position = UDim2.new(0,0,0,34), BackgroundColor3 = theme.Sidebar})
    if rounded then new("UICorner", {Parent = sidebar, CornerRadius = UDim.new(0,8)}) end
    local divider = new("Frame", {Parent = main, Size = UDim2.new(0,6,1,-34), Position = UDim2.new(0,sidebarWidth,0,34), BackgroundColor3 = theme.Divider})
    local content = new("Frame", {Parent = main, Size = UDim2.new(1,-sidebarWidth-6,1,-34), Position = UDim2.new(0,sidebarWidth+6,0,34), BackgroundColor3 = theme.Background})
    if rounded then new("UICorner", {Parent = content, CornerRadius = UDim.new(0,8)}) end

    -- Internal layout holders
    local sideLayout = new("UIListLayout", {Parent = sidebar, SortOrder = Enum.SortOrder.LayoutOrder, Padding = UDim.new(0,8)})
    sideLayout.Padding = UDim.new(0,8)
    sideLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center

    local contentLayout = new("UIListLayout", {Parent = content, SortOrder = Enum.SortOrder.LayoutOrder, Padding = UDim.new(0,8)})
    contentLayout.Padding = UDim.new(0,8)

    -- draggable
    makeDraggable(main, titleBar)

    -- Window state
    local state = {
        Minimized = false,
        Maximized = false,
        PrevSize = main.Size,
        PrevPos = main.Position
    }

    -- Toggle/minimize/maximize/close functions
    local function minimize()
        if state.Minimized then return end
        state.PrevSize = main.Size
        state.PrevPos = main.Position
        TweenService:Create(content, TweenInfo.new(0.18), {Size = UDim2.new(1,0,0,0)}):Play()
        content.Visible = false
        state.Minimized = true
    end
    local function maximize()
        if state.Maximized then
            -- restore
            main:TweenSizeAndPosition(state.PrevSize, state.PrevPos, Enum.EasingDirection.Out, Enum.EasingStyle.Quad, 0.18, true)
            state.Maximized = false
        else
            state.PrevSize = main.Size
            state.PrevPos = main.Position
            local fullSize = UDim2.new(0.85,0,0.75,0)
            main:TweenSizeAndPosition(fullSize, UDim2.new(0.5,-(fullSize.X.Offset)/2,0.5,-(fullSize.Y.Offset)/2), Enum.EasingDirection.Out, Enum.EasingStyle.Quad, 0.18, true)
            state.Maximized = true
        end
    end
    local function close()
        if screen and screen.Parent then
            screen:Destroy()
        end
    end
    local function toggleVisibility()
        if not main or not main.Parent then return end
        main.Visible = not main.Visible
    end

    btnMin.MouseButton1Click:Connect(function() if state.Minimized then
            -- restore
            content.Visible = true
            TweenService:Create(content, TweenInfo.new(0.18), {Size = UDim2.new(1,0,1,0)}):Play()
            state.Minimized = false
        else
            minimize()
        end
    end)
    btnMax.MouseButton1Click:Connect(maximize)
    btnClose.MouseButton1Click:Connect(close)

    -- floating toggle button on screen
    local toggleBtn = createToggleButton(screen, theme)
    toggleBtn.MouseButton1Click:Connect(function()
        toggleVisibility()
    end)

    -- API: create tabs and sections
    local win = {Gui = screen, Main = main, Sidebar = sidebar, Content = content, Tabs = {}, _order = {}}

    function win:CreateTab(name)
        local btn = new("TextButton", {Parent = sidebar, Size = UDim2.new(1,-20,0,34), Text = name, BackgroundColor3 = theme.Sidebar, TextColor3 = theme.Text, Font = Enum.Font.Gotham, TextSize = 14})
        new("UICorner", {Parent = btn, CornerRadius = UDim.new(0,8)})

        local tabFrame = new("ScrollingFrame", {Parent = content, Size = UDim2.new(1,0,1,0), CanvasSize = UDim2.new(0,0), ScrollBarThickness = 6, BackgroundTransparency = 1, Visible = false})
        new("UIListLayout", {Parent = tabFrame, SortOrder = Enum.SortOrder.LayoutOrder, Padding = UDim.new(0,8)})

        btn.MouseButton1Click:Connect(function()
            for _,t in pairs(win.Tabs) do t.Frame.Visible = false; TweenService:Create(t.Btn, TweenInfo.new(0.18), {BackgroundColor3 = theme.Sidebar}):Play() end
            tabFrame.Visible = true
            TweenService:Create(btn, TweenInfo.new(0.18), {BackgroundColor3 = theme.Accent}):Play()
        end)

        win.Tabs[name] = {Btn = btn, Frame = tabFrame}
        table.insert(win._order, name)

        -- if first tab, activate
        if #win._order == 1 then
            btn:MouseButton1Click()
        end

        local tabAPI = {}
        function tabAPI:CreateSection(title)
            local section = new("Frame", {Parent = tabFrame, Size = UDim2.new(1,0,0,100), BackgroundColor3 = theme.Sidebar})
            new("UICorner", {Parent = section, CornerRadius = UDim.new(0,8)})
            local header = new("TextLabel", {Parent = section, Size = UDim2.new(1,-16,0,26), Position = UDim2.new(0,8,0,6), BackgroundTransparency = 1, Text = title, TextColor3 = theme.Text, Font = Enum.Font.GothamBold, TextSize = 14, TextXAlignment = Enum.TextXAlignment.Left})
            local holder = new("Frame", {Parent = section, Size = UDim2.new(1,-16,1,-34), Position = UDim2.new(0,8,0,30), BackgroundTransparency = 1})
            new("UIListLayout", {Parent = holder, SortOrder = Enum.SortOrder.LayoutOrder, Padding = UDim.new(0,8)})

            local sectionAPI = {}
            function sectionAPI:Button(text, callback)
                local b = new("TextButton", {Parent = holder, Size = UDim2.new(1,0,0,32), Text = text, BackgroundColor3 = theme.Background, TextColor3 = theme.Text, Font = Enum.Font.Gotham, TextSize = 14})
                new("UICorner", {Parent = b, CornerRadius = UDim.new(0,6)})
                b.MouseButton1Click:Connect(function() pcall(callback) end)
                return b
            end
            function sectionAPI:Toggle(text, state, callback)
                local container = new("Frame", {Parent = holder, Size = UDim2.new(1,0,0,32), BackgroundTransparency = 1})
                local label = new("TextLabel", {Parent = container, Size = UDim2.new(1,-50,1,0), BackgroundTransparency = 1, Text = text, TextColor3 = theme.Text, Font = Enum.Font.Gotham, TextSize = 14, TextXAlignment = Enum.TextXAlignment.Left})
                local toggleBtn = new("TextButton", {Parent = container, Size = UDim2.new(0,36,0,20), Position = UDim2.new(1,-40,0,6), BackgroundColor3 = state and theme.Accent or Color3.fromRGB(80,80,80), Text = ""})
                new("UICorner", {Parent = toggleBtn, CornerRadius = UDim.new(0,10)})
                local s = state
                toggleBtn.MouseButton1Click:Connect(function()
                    s = not s
                    toggleBtn.BackgroundColor3 = s and theme.Accent or Color3.fromRGB(80,80,80)
                    pcall(callback, s)
                end)
                return {Get = function() return s end}
            end
            function sectionAPI:Slider(text, min, max, default, callback)
                min = min or 0; max = max or 100; default = default or min
                local container = new("Frame", {Parent = holder, Size = UDim2.new(1,0,0,44), BackgroundTransparency = 1})
                local label = new("TextLabel", {Parent = container, Size = UDim2.new(1,0,0,18), BackgroundTransparency = 1, Text = text.." - "..tostring(default), TextColor3 = theme.Text, Font = Enum.Font.Gotham, TextSize = 13, TextXAlignment = Enum.TextXAlignment.Left})
                local bar = new("Frame", {Parent = container, Size = UDim2.new(1,0,0,12), Position = UDim2.new(0,0,0,24), BackgroundColor3 = Color3.fromRGB(60,60,60)})
                new("UICorner", {Parent = bar, CornerRadius = UDim.new(0,6)})
                local fill = new("Frame", {Parent = bar, Size = UDim2.new((default-min)/(max-min),0,1,0), BackgroundColor3 = theme.Accent})
                new("UICorner", {Parent = fill, CornerRadius = UDim.new(0,6)})

                local holding = false
                bar.InputBegan:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 then holding = true end end)
                bar.InputEnded:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 then holding = false end end)
                UserInputService.InputChanged:Connect(function(input)
                    if holding and input.UserInputType == Enum.UserInputType.MouseMovement then
                        local rel = math.clamp((input.Position.X - bar.AbsolutePosition.X) / bar.AbsoluteSize.X, 0, 1)
                        fill.Size = UDim2.new(rel,0,1,0)
                        local value = min + (max-min) * rel
                        label.Text = text.." - "..math.floor(value*100)/100
                        pcall(callback, value)
                    end
                end)
                return {Get = function() return min + (max-min) * fill.Size.X.Scale end}
            end
            function sectionAPI:TextBox(placeholder, callback)
                local box = new("TextBox", {Parent = holder, Size = UDim2.new(1,0,0,28), Text = "", PlaceholderText = placeholder or "...", Font = Enum.Font.Gotham, TextSize = 14, BackgroundColor3 = theme.Background, TextColor3 = theme.Text})
                new("UICorner", {Parent = box, CornerRadius = UDim.new(0,6)})
                box.FocusLost:Connect(function(enter) if enter then pcall(callback, box.Text) end end)
                return box
            end
            function sectionAPI:Dropdown(text, list, callback)
                local container = new("Frame", {Parent = holder, Size = UDim2.new(1,0,0,30), BackgroundTransparency = 1})
                local label = new("TextLabel", {Parent = container, Size = UDim2.new(1,-30,1,0), BackgroundTransparency = 1, Text = text, TextColor3 = theme.Text, Font = Enum.Font.Gotham, TextSize = 14, TextXAlignment = Enum.TextXAlignment.Left})
                local btn = new("TextButton", {Parent = container, Size = UDim2.new(0,24,0,24), Position = UDim2.new(1,-26,0,3), Text = ">", BackgroundColor3 = theme.Sidebar})
                new("UICorner", {Parent = btn, CornerRadius = UDim.new(0,6)})
                local expanded = false
                local listFrame
                btn.MouseButton1Click:Connect(function()
                    expanded = not expanded
                    if expanded then
                        listFrame = new("Frame", {Parent = container, Size = UDim2.new(1,0,0,#list * 28), Position = UDim2.new(0,0,1,6), BackgroundColor3 = theme.Sidebar})
                        new("UICorner", {Parent = listFrame, CornerRadius = UDim.new(0,6)})
                        for i,v in ipairs(list) do
                            local item = new("TextButton", {Parent = listFrame, Size = UDim2.new(1,0,0,28), Position = UDim2.new(0,0,0,(i-1)*28), Text = v, BackgroundColor3 = theme.Background, TextColor3 = theme.Text})
                            item.MouseButton1Click:Connect(function() pcall(callback, v); listFrame:Destroy(); expanded = false end)
                        end
                    else
                        if listFrame then listFrame:Destroy() end
                    end
                end)
                return {Label = label, Button = btn}
            end

            return sectionAPI
        end

        return tabAPI
    end

    -- expose window control methods
    function win:Minimize() minimize() end
    function win:Maximize() maximize() end
    function win:Close() close() end
    function win:Toggle() toggleVisibility() end

    table.insert(Vix._windows, win)
    return win
end

-- Global helper to destroy all
function Vix:DestroyAll()
    for _,w in pairs(self._windows) do
        pcall(function() if w.Gui and w.Gui.Parent then w.Gui:Destroy() end end)
    end
    self._windows = {}
end

return Vix
