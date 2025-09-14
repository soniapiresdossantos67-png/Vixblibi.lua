-- Vixblibi.lua
-- Biblioteca de UI com abas, seções, botões, minimizar/maximizar/fechar e botão flutuante

local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")

local Vix = {}
Vix.windows = {}

-- Função utilitária para criar instâncias
local function make(class, props)
    local obj = Instance.new(class)
    for k, v in pairs(props) do
        obj[k] = v
    end
    return obj
end

-- Função principal para criar a janela
function Vix:CreateWindow(title)
    local ScreenGui = make("ScreenGui", {
        Name = "VixUI",
        ResetOnSpawn = false,
        Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")
    })

    -- Janela principal
    local Window = make("Frame", {
        Parent = ScreenGui,
        BackgroundColor3 = Color3.fromRGB(35, 35, 35),
        BorderSizePixel = 0,
        Size = UDim2.new(0, 500, 0, 300),
        Position = UDim2.new(0.5, -250, 0.5, -150),
        Visible = true
    })
    Window.ClipsDescendants = true
    Window.Active = true
    Window.Draggable = true

    -- Barra superior
    local TopBar = make("Frame", {
        Parent = Window,
        BackgroundColor3 = Color3.fromRGB(25, 25, 25),
        BorderSizePixel = 0,
        Size = UDim2.new(1, 0, 0, 30)
    })

    local TitleLabel = make("TextLabel", {
        Parent = TopBar,
        BackgroundTransparency = 1,
        Size = UDim2.new(1, -90, 1, 0),
        Position = UDim2.new(0, 10, 0, 0),
        Font = Enum.Font.SourceSansBold,
        Text = title or "Vix UI",
        TextColor3 = Color3.new(1, 1, 1),
        TextSize = 18,
        TextXAlignment = Enum.TextXAlignment.Left
    })

    -- Botões do topo
    local CloseButton = make("TextButton", {
        Parent = TopBar,
        Size = UDim2.new(0, 30, 1, 0),
        Position = UDim2.new(1, -30, 0, 0),
        BackgroundColor3 = Color3.fromRGB(200, 50, 50),
        Text = "X",
        TextColor3 = Color3.new(1, 1, 1),
        Font = Enum.Font.SourceSansBold,
        TextSize = 16
    })

    local MinButton = make("TextButton", {
        Parent = TopBar,
        Size = UDim2.new(0, 30, 1, 0),
        Position = UDim2.new(1, -60, 0, 0),
        BackgroundColor3 = Color3.fromRGB(255, 170, 0),
        Text = "-",
        TextColor3 = Color3.new(1, 1, 1),
        Font = Enum.Font.SourceSansBold,
        TextSize = 16
    })

    local MaxButton = make("TextButton", {
        Parent = TopBar,
        Size = UDim2.new(0, 30, 1, 0),
        Position = UDim2.new(1, -90, 0, 0),
        BackgroundColor3 = Color3.fromRGB(0, 170, 255),
        Text = "+",
        TextColor3 = Color3.new(1, 1, 1),
        Font = Enum.Font.SourceSansBold,
        TextSize = 16
    })

    -- Conteúdo
    local TabHolder = make("Frame", {
        Parent = Window,
        BackgroundColor3 = Color3.fromRGB(30, 30, 30),
        BorderSizePixel = 0,
        Position = UDim2.new(0, 0, 0, 30),
        Size = UDim2.new(0, 120, 1, -30)
    })

    local ContentFrame = make("Frame", {
        Parent = Window,
        BackgroundColor3 = Color3.fromRGB(45, 45, 45),
        BorderSizePixel = 0,
        Position = UDim2.new(0, 120, 0, 30),
        Size = UDim2.new(1, -120, 1, -30)
    })

    -- Botão flutuante
    local MenuButton = make("TextButton", {
        Parent = ScreenGui,
        BackgroundColor3 = Color3.fromRGB(0, 170, 127),
        Size = UDim2.new(0, 100, 0, 40),
        Position = UDim2.new(0.05, 0, 0.85, 0),
        Font = Enum.Font.SourceSansBold,
        Text = "Menu",
        TextSize = 18,
        TextColor3 = Color3.new(1, 1, 1)
    })

    local minimized = false
    local maximized = false

    MinButton.MouseButton1Click:Connect(function()
        if not minimized then
            TweenService:Create(Window, TweenInfo.new(0.3), {Size = UDim2.new(0, 500, 0, 30)}):Play()
            minimized = true
        else
            TweenService:Create(Window, TweenInfo.new(0.3), {Size = UDim2.new(0, 500, 0, 300)}):Play()
            minimized = false
        end
    end)

    MaxButton.MouseButton1Click:Connect(function()
        if not maximized then
            TweenService:Create(Window, TweenInfo.new(0.3), {Size = UDim2.new(1, -50, 1, -50), Position = UDim2.new(0, 25, 0, 25)}):Play()
            maximized = true
        else
            TweenService:Create(Window, TweenInfo.new(0.3), {Size = UDim2.new(0, 500, 0, 300), Position = UDim2.new(0.5, -250, 0.5, -150)}):Play()
            maximized = false
        end
    end)

    CloseButton.MouseButton1Click:Connect(function()
        Window.Visible = false
    end)

    MenuButton.MouseButton1Click:Connect(function()
        Window.Visible = not Window.Visible
    end)

    -- Funções para criar Tabs e Seções
    local WindowAPI = {}

    function WindowAPI:CreateTab(tabName)
        local TabButton = make("TextButton", {
            Parent = TabHolder,
            Size = UDim2.new(1, 0, 0, 30),
            BackgroundColor3 = Color3.fromRGB(50, 50, 50),
            BorderSizePixel = 0,
            Text = tabName,
            TextColor3 = Color3.new(1, 1, 1),
            Font = Enum.Font.SourceSans,
            TextSize = 16
        })

        local TabContent = make("ScrollingFrame", {
            Parent = ContentFrame,
            BackgroundTransparency = 1,
            Size = UDim2.new(1, 0, 1, 0),
            CanvasSize = UDim2.new(0, 0, 0, 0),
            Visible = false
        })

        local UIListLayout = make("UIListLayout", {
            Parent = TabContent,
            Padding = UDim.new(0, 5),
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
                Size = UDim2.new(1, -10, 0, 40)
            })

            local SectionLabel = make("TextLabel", {
                Parent = Section,
                BackgroundTransparency = 1,
                Size = UDim2.new(1, 0, 0, 20),
                Font = Enum.Font.SourceSansBold,
                Text = sectionName,
                TextSize = 16,
                TextColor3 = Color3.new(1, 1, 1),
                TextXAlignment = Enum.TextXAlignment.Left
            })

            local SectionAPI = {}

            function SectionAPI:Button(text, callback)
                local Btn = make("TextButton", {
                    Parent = Section,
                    BackgroundColor3 = Color3.fromRGB(80, 80, 80),
                    Size = UDim2.new(1, -10, 0, 30),
                    Position = UDim2.new(0, 5, 0, 20),
                    Text = text,
                    TextColor3 = Color3.new(1, 1, 1),
                    Font = Enum.Font.SourceSans,
                    TextSize = 16
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
