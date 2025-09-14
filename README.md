-- Vixblibi.lua (versão compacta e corrigida)
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")

local Vix = {}
Vix.windows = {}

local function make(class, props)
    local obj = Instance.new(class)
    for k, v in pairs(props) do
        obj[k] = v
    end
    return obj
end

function Vix:CreateWindow(title)
    local ScreenGui = make("ScreenGui", {
        Name = "VixUI",
        ResetOnSpawn = false,
        Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")
    })

    -- Janela principal (compacta)
    local Window = make("Frame", {
        Parent = ScreenGui,
        BackgroundColor3 = Color3.fromRGB(35, 35, 35),
        BorderSizePixel = 0,
        Size = UDim2.new(0, 350, 0, 220),
        Position = UDim2.new(0.5, -175, 0.5, -110),
        Visible = true
    })
    Window.ClipsDescendants = true

    -- Barra superior
    local TopBar = make("Frame", {
        Parent = Window,
        BackgroundColor3 = Color3.fromRGB(25, 25, 25),
        BorderSizePixel = 0,
        Size = UDim2.new(1, 0, 0, 28)
    })

    local TitleLabel = make("TextLabel", {
        Parent = TopBar,
        BackgroundTransparency = 1,
        Size = UDim2.new(1, -90, 1, 0),
        Position = UDim2.new(0, 10, 0, 0),
        Font = Enum.Font.SourceSansBold,
        Text = title or "Vix UI",
        TextColor3 = Color3.new(1, 1, 1),
        TextSize = 16,
        TextXAlignment = Enum.TextXAlignment.Left
    })

    -- Botões do topo
    local function makeTopBtn(txt, offset, color)
        return make("TextButton", {
            Parent = TopBar,
            Size = UDim2.new(0, 26, 1, -4),
            Position = UDim2.new(1, offset, 0, 2),
            BackgroundColor3 = color,
            Text = txt,
            TextColor3 = Color3.new(1, 1, 1),
            Font = Enum.Font.SourceSansBold,
            TextSize = 14
        })
    end

    local CloseButton = makeTopBtn("X", -28, Color3.fromRGB(200, 50, 50))
    local MinButton = makeTopBtn("-", -56, Color3.fromRGB(255, 170, 0))
    local MaxButton = makeTopBtn("+", -84, Color3.fromRGB(0, 170, 255))

    -- Conteúdo
    local TabHolder = make("Frame", {
        Parent = Window,
        BackgroundColor3 = Color3.fromRGB(30, 30, 30),
        BorderSizePixel = 0,
        Position = UDim2.new(0, 0, 0, 28),
        Size = UDim2.new(0, 100, 1, -28)
    })

    local ContentFrame = make("Frame", {
        Parent = Window,
        BackgroundColor3 = Color3.fromRGB(45, 45, 45),
        BorderSizePixel = 0,
        Position = UDim2.new(0, 100, 0, 28),
        Size = UDim2.new(1, -100, 1, -28)
    })

    -- 🔵 Botão bolinha flutuante
    local MenuButton = make("TextButton", {
        Parent = ScreenGui,
        BackgroundColor3 = Color3.fromRGB(0, 170, 127),
        Size = UDim2.new(0, 50, 0, 50),
        Position = UDim2.new(0.05, 0, 0.8, 0),
        Font = Enum.Font.SourceSansBold,
        Text = "≡",
        TextSize = 22,
        TextColor3 = Color3.new(1, 1, 1)
    })
    MenuButton.TextScaled = true
    MenuButton.AutoButtonColor = true
    make("UICorner", {Parent = MenuButton, CornerRadius = UDim.new(1,0)})

    -- Minimizar, maximizar, fechar
    local minimized = false
    local maximized = false

    MinButton.MouseButton1Click:Connect(function()
        if not minimized then
            TweenService:Create(Window, TweenInfo.new(0.3), {Size = UDim2.new(0, 350, 0, 28)}):Play()
            minimized = true
        else
            TweenService:Create(Window, TweenInfo.new(0.3), {Size = UDim2.new(0, 350, 0, 220)}):Play()
            minimized = false
        end
    end)

    MaxButton.MouseButton1Click:Connect(function()
        if not maximized then
            TweenService:Create(Window, TweenInfo.new(0.3), {
                Size = UDim2.new(1, -50, 1, -50), 
                Position = UDim2.new(0, 25, 0, 25)
            }):Play()
            maximized = true
        else
            TweenService:Create(Window, TweenInfo.new(0.3), {
                Size = UDim2.new(0, 350, 0, 220), 
                Position = UDim2.new(0.5, -175, 0.5, -110)
            }):Play()
            maximized = false
        end
    end)

    CloseButton.MouseButton1Click:Connect(function()
        Window.Visible = false
    end)

    MenuButton.MouseButton1Click:Connect(function()
        Window.Visible = not Window.Visible
    end)

    -- Drag só na TopBar
    local dragging, dragStart, startPos
    TopBar.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStart = input.Position
            startPos = Window.Position
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
            Window.Position = UDim2.new(
                startPos.X.Scale, startPos.X.Offset + delta.X,
                startPos.Y.Scale, startPos.Y.Offset + delta.Y
            )
        end
    end)

    -- API
    local WindowAPI = {}

    function WindowAPI:CreateTab(tabName)
        local TabButton = make("TextButton", {
            Parent = TabHolder,
            Size = UDim2.new(1, 0, 0, 28),
            BackgroundColor3 = Color3.fromRGB(50, 50, 50),
            BorderSizePixel = 0,
            Text = tabName,
            TextColor3 = Color3.new(1, 1, 1),
            Font = Enum.Font.SourceSans,
            TextSize = 14
        })

        local TabContent = make("ScrollingFrame", {
            Parent = ContentFrame,
            BackgroundTransparency = 1,
            Size = UDim2.new(1, 0, 1, 0),
            CanvasSize = UDim2.new(0, 0, 0, 0),
            Visible = false
        })

        make("UIListLayout", {
            Parent = TabContent,
            Padding = UDim.new(0, 4),
            SortOrder = Enum.SortOrder.LayoutOrder
        })

        TabButton.MouseButton1Click:Connect(function()
            for _, frame in pairs(ContentFrame:GetChildren()) do
                if frame:IsA("ScrollingFrame") then
                    frame.Visible = false
                end
            end
            TabContent.Visible = true
        end)

        local TabAPI = {}

        function TabAPI:CreateSection(sectionName)
            local Section = make("Frame", {
                Parent = TabContent,
                BackgroundColor3 = Color3.fromRGB(60, 60, 60),
                Size = UDim2.new(1, -8, 0, 60)
            })

            make("UIListLayout", {
                Parent = Section,
                Padding = UDim.new(0, 4),
                SortOrder = Enum.SortOrder.LayoutOrder
            })

            make("TextLabel", {
                Parent = Section,
                BackgroundTransparency = 1,
                Size = UDim2.new(1, 0, 0, 18),
                Font = Enum.Font.SourceSansBold,
                Text = sectionName,
                TextSize = 14,
                TextColor3 = Color3.new(1, 1, 1),
                TextXAlignment = Enum.TextXAlignment.Left
            })

            local SectionAPI = {}

            function SectionAPI:Button(text, callback)
                local Btn = make("TextButton", {
                    Parent = Section,
                    BackgroundColor3 = Color3.fromRGB(80, 80, 80),
                    Size = UDim2.new(1, -8, 0, 24),
                    Text = text,
                    TextColor3 = Color3.new(1, 1, 1),
                    Font = Enum.Font.SourceSans,
                    TextSize = 14
                })
                Btn.MouseButton1Click:Connect(callback)
            end

            return SectionAPI
        end

        return TabAPI
    end

    return WindowAPI
end

return Vix
