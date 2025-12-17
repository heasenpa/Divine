local Players = game:GetService("Players")
local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

local SCRIPT_TAG = "[PhoneAutoSelect]"
local clicked = false

local function log(msg)
    print(SCRIPT_TAG, msg)
end

-- Click không phụ thuộc kích thước màn hình
local function clickButton(btn)
    if not btn or clicked then return end
    clicked = true

    -- Ưu tiên Activated (ổn định nhất)
    pcall(function()
        firesignal(btn.Activated)
    end)

    -- Fallback MouseButton1Click
    pcall(function()
        firesignal(btn.MouseButton1Click)
    end)

    -- VirtualInput chỉ dùng nếu 2 cách trên fail
    task.delay(0.05, function()
        if not btn or not btn.Parent then return end
        local vim = game:GetService("VirtualInputManager")
        local center = btn.AbsolutePosition + (btn.AbsoluteSize / 2)

        vim:SendMouseButtonEvent(center.X, center.Y, 0, true, game, 1)
        vim:SendMouseButtonEvent(center.X, center.Y, 0, false, game, 1)
    end)
end

local function trySelectPhone(deviceSelect)
    if clicked then return end

    local container = deviceSelect:FindFirstChild("Container")
    local phone = container and container:FindFirstChild("Phone")
    local button = phone and phone:FindFirstChildWhichIsA("GuiButton")

    if not button then return end
    if not button.Visible or not button.Active then return end

    log("Auto clicking Phone button")
    clickButton(button)
end

local function watchDeviceSelect()
    local deviceSelect = playerGui:FindFirstChild("DeviceSelect")

    if deviceSelect then
        if deviceSelect.Enabled then
            trySelectPhone(deviceSelect)
        end

        deviceSelect:GetPropertyChangedSignal("Enabled"):Connect(function()
            if deviceSelect.Enabled then
                task.wait()
                trySelectPhone(deviceSelect)
            end
        end)
    end

    playerGui.ChildAdded:Connect(function(child)
        if child.Name == "DeviceSelect" then
            log("DeviceSelect added")
            watchDeviceSelect()
        end
    end)
end

watchDeviceSelect()
