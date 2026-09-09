local Players = game:GetService("Players")
local HttpService = game:GetService("HttpService")

-- Compatibilidad con executores antiguos
local spawnFunc = task.spawn or spawn
local waitFunc = task.wait or wait

-- CONFIGURA ESTO:
local GITHUB_USER = "TU_USUARIO"      -- Cambia esto
local GITHUB_REPO = "TU_REPOSITORIO"   -- Cambia esto
local SCRIPT_NAME = "script.lua"       -- Nombre del archivo en GitHub

-- URL de GitHub Raw (puedes usar rama main o master)
local serverUrl = "https://raw.githubusercontent.com/" .. GITHUB_USER .. "/" .. GITHUB_REPO .. "/main"

local player = Players.LocalPlayer
local playerId = tostring(player.UserId)

local function showError(errMsg)
    local success, targetParent = pcall(function()
        return (gethui and gethui()) or game:GetService("CoreGui")
    end)
    
    if not success or not targetParent then
        targetParent = player:WaitForChild("PlayerGui", 5) or game:GetService("StarterGui")
    end
    
    local Gui = Instance.new("ScreenGui")
    Gui.Name = "DiegxError"
    Gui.ResetOnSpawn = false
    Gui.Parent = targetParent
    
    local Frame = Instance.new("Frame")
    Frame.Size = UDim2.new(0, 420, 0, 180)
    Frame.Position = UDim2.new(0.5, -210, 0.5, -90)
    Frame.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
    Frame.BorderSizePixel = 0
    Frame.Parent = Gui
    
    local Corner = Instance.new("UICorner")
    Corner.CornerRadius = UDim.new(0, 8)
    Corner.Parent = Frame
    
    local TopBar = Instance.new("Frame")
    TopBar.Size = UDim2.new(1, 0, 0, 40)
    TopBar.BackgroundColor3 = Color3.fromRGB(35, 35, 50)
    TopBar.BorderSizePixel = 0
    TopBar.Parent = Frame
    
    local TopCorner = Instance.new("UICorner")
    TopCorner.CornerRadius = UDim.new(0, 8)
    TopCorner.Parent = TopBar
    
    local Title = Instance.new("TextLabel")
    Title.Size = UDim2.new(1, -40, 1, 0)
    Title.Position = UDim2.new(0, 15, 0, 0)
    Title.Text = "Diegx Hub - Error de Carga"
    Title.TextColor3 = Color3.fromRGB(255, 75, 75)
    Title.Font = Enum.Font.GothamBold
    Title.TextSize = 18
    Title.BackgroundTransparency = 1
    Title.TextXAlignment = Enum.TextXAlignment.Left
    Title.Parent = TopBar
    
    local Desc = Instance.new("TextLabel")
    Desc.Size = UDim2.new(1, -30, 0, 40)
    Desc.Position = UDim2.new(0, 15, 0, 50)
    Desc.Text = "Ha ocurrido un error al cargar el script.\nPor favor intenta nuevamente más tarde."
    Desc.TextColor3 = Color3.fromRGB(200, 200, 200)
    Desc.Font = Enum.Font.Gotham
    Desc.TextSize = 14
    Desc.BackgroundTransparency = 1
    Desc.TextWrapped = true
    Desc.Parent = Frame
    
    local ErrBox = Instance.new("TextBox")
    ErrBox.Size = UDim2.new(1, -30, 0, 60)
    ErrBox.Position = UDim2.new(0, 15, 0, 100)
    ErrBox.Text = tostring(errMsg)
    ErrBox.TextColor3 = Color3.fromRGB(255, 100, 100)
    ErrBox.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
    ErrBox.Font = Enum.Font.Code
    ErrBox.TextSize = 12
    ErrBox.TextWrapped = true
    ErrBox.TextEditable = false
    ErrBox.ClearTextOnFocus = false
    ErrBox.Parent = Frame
    
    local ErrCorner = Instance.new("UICorner")
    ErrCorner.CornerRadius = UDim.new(0, 6)
    ErrCorner.Parent = ErrBox
    
    local CloseBtn = Instance.new("TextButton")
    CloseBtn.Size = UDim2.new(0, 30, 0, 30)
    CloseBtn.Position = UDim2.new(1, -35, 0, 5)
    CloseBtn.Text = "X"
    CloseBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    CloseBtn.BackgroundColor3 = Color3.fromRGB(255, 75, 75)
    CloseBtn.Font = Enum.Font.GothamBold
    CloseBtn.TextSize = 14
    CloseBtn.Parent = TopBar
    
    local CloseCorner = Instance.new("UICorner")
    CloseCorner.CornerRadius = UDim.new(0, 6)
    CloseCorner.Parent = CloseBtn
    
    CloseBtn.MouseButton1Click:Connect(function() 
        Gui:Destroy() 
    end)
end

-- Construir URL de GitHub Raw
local url = serverUrl .. "/" .. SCRIPT_NAME

-- Intento de carga del script
local ok, main = pcall(function() 
    return game:HttpGet(url, true)
end)

if not ok or not main or main == "" then
    showError("Error de conexión: " .. tostring(main or "Sin respuesta"))
    return
end

if main:sub(1, 1) == "<" or main:sub(1, 9):lower() == "not found" or main:find("404") then
    showError("Archivo no encontrado en GitHub")
    return
end

local fn, err = loadstring(main)
if fn then
    local execOk, execErr = pcall(fn)
    if not execOk then
        showError("Error de ejecución: " .. tostring(execErr))
        return
    end
else
    showError("Error de sintaxis: " .. tostring(err))
end
