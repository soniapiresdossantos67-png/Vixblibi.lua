--// Vixblibi.lua
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local Players = game:GetService("Players")
local player = Players.LocalPlayer

local Vix = {}
Vix.windows = {}

-- Função auxiliar
local function make(class, props)
    local obj = Instance.new(class)
    for k,v in pairs(props or {}) do
        obj[k] = v
    end
    return obj
end

-- Tema padrão
local defaultTheme = {
    Background = Color3.fromRGB(25,25,35),
    Sidebar    = Color3.fromRGB(18,18,25),
    Accent     = Color3.fromRGB(0,170,140),
    Text       = Color3.fromRGB(255,255,255),
}

-- Criar janela
function Vix:CreateWindow(opts)
    opts = opts or {}
    local theme = opts.Theme or defaultTheme
    local size = opts.Size or UDim2.new(0,500,0,300)

    -- ScreenGui
    local gui = make("ScreenGui", {
        Name = "VixUI",
        Parent = game:GetService("CoreGui"),
        ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    })

    -- Botão flutuante
    local toggleBtn = make("TextButton", {
        Parent = gui,
        Text = "📂 Menu",
        Size = UDim2.new(0,100,0,35),
        Position = UDim2.new(0,20,1,-55),
        BackgroundColor3 = theme.Accent,
        TextColor3 = Color3.new(1,1,1),
        Font = Enum.Font.GothamBold,
        TextSize = 14,
        AutoButtonColor = true
    })
    toggleBtn.AnchorPoint = Vector2.new(0,1)

    -- Frame principal
    local window = make("Frame", {
        Parent = gui,
        Size = size,
        Position = UDim2.new(0.5,-size.X.Offset/2,0.5,-size.Y.Offset/2),
        BackgroundColor3 = theme.Background,
        BorderSizePixel = 0,
        Visible = false,
    })
    window.ClipsDescendants = true
    window.Active = true
    window.Draggable = false

    -- Cantos arredondados
    make("UICorner", {Parent = window, CornerRadius = UDim.new(0,12)})

    -- Barra superior
    local titleBar = make("Frame", {
        Parent = window,
        Size = UDim2.new(1,0,0,30),
        BackgroundColor3 = theme.Sidebar,
        BorderSizePixel = 0,
    })
    make("UICorner", {Parent = titleBar, CornerRadius = UDim.new(0,12)})

    -- Título
    make("TextLabel", {
        Parent = titleBar,
        Text = opts.Title or "Vixblibi",
        Size = UDim2.new(1,-90,1,0),
        Position = UDim2.new(0,10,0,0),
        BackgroundTransparency = 1,
        TextColor3 = theme.Text,
        TextXAlignment = Enum.TextXAlignment.Left,
        Font = Enum.Font.GothamBold,
        TextSize = 14
    })

    -- Botões topo
    local function createTopBtn(txt, posX)
        local b = make("TextButton", {
            Parent = titleBar,
            Text = txt,
            Size = UDim2.new(0,25,0,25),
            Position = UDim2.new(1,-posX,0.5,-12),
            BackgroundColor3 = theme.Accent,
            TextColor3 = Color3.new(1,1,1),
            Font = Enum.Font.GothamBold,
            TextSize = 12,
        })
        make("UICorner", {Parent=b, CornerRadius=UDim.new(0,6)})
        return b
    end
    local btnMin = createTopBtn("-", 80)
    local btnMax = createTopBtn("□", 50)
    local btnClose = createTopBtn("X", 20)

    -- Sidebar
    local sidebar = make("Frame", {
        Parent = window,
        Size = UDim2.new(0,120,1,-30),
        Position = UDim2.new(0,0,0,30),
        BackgroundColor3 = theme.Sidebar,
        BorderSizePixel = 0
    })

    -- Conteúdo
    local content = make("Frame", {
        Parent = window,
        Size = UDim2.new(1,-120,1,-30),
        Position = UDim2.new(0,120,0,30),
        BackgroundColor3 = theme.Background,
        BorderSizePixel = 0
    })

    -- API Window
    local api = {}
    api.Gui = gui
    api.Window = window
    api.Sidebar = sidebar
    api.Content = content

    function api:Toggle()
        window.Visible = not window.Visible
    end

    function api:Close()
        window.Visible = false
    end

    function api:Minimize()
        TweenService:Create(window, TweenInfo.new(0.25), {Size=UDim2.new(0,200,0,30)}):Play()
    end

    function api:Maximize()
        TweenService:Create(window, TweenInfo.new(0.25), {Size=size}):Play()
    end

    -- Toggle btn
    toggleBtn.MouseButton1Click:Connect(function()
        api:Toggle()
    end)

    -- Top buttons
    btnMin.MouseButton1Click:Connect(function() api:Minimize() end)
    btnMax.MouseButton1Click:Connect(function() api:Maximize() end)
    btnClose.MouseButton1Click:Connect(function() api:Close() end)

    -- Movimento da janela
    local dragging, dragStart, startPos
    titleBar.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStart = input.Position
            startPos = window.Position
        end
    end)
    titleBar.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = false
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
            local delta = input.Position - dragStart
            window.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)

    Vix.windows[#Vix.windows+1] = api
    return api
end

return Vix
