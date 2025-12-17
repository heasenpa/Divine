local Players = game:GetService("Players")
local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

local SCRIPT_TAG = "[PhoneAutoSelect]"

local function log(msg)
    print(SCRIPT_TAG, msg)
end

-- Fire click an toàn hơn
local function clickButton(btn)
    if not btn then return end

    -- Ưu tiên firesignal
    pcall(function()
        firesignal(btn.MouseButton1Click)
    end)

    -- Fallback VirtualInput
    pcall(function()
        local VirtualInputManager = game:GetService("VirtualInputManager")
        local pos = btn.AbsolutePosition + (btn.AbsoluteSize / 2)

        VirtualInputManager:SendMouseButtonEvent(
            pos.X, pos.Y, 0, true, game, 1
        )
        VirtualInputManager:SendMouseButtonEvent(
            pos.X, pos.Y, 0, false, game, 1
        )
    end)
end

-- Xử lý khi DeviceSelect được bật
local function trySelectPhone(deviceSelect)
    local container = deviceSelect:FindFirstChild("Container")
    if not container then
        log("Container not found")
        return
    end

    local phone = container:FindFirstChild("Phone")
    if not phone then
        log("Phone not found")
        return
    end

    local button = phone:FindFirstChildWhichIsA("GuiButton")
    if not button then
        log("Button not found")
        return
    end

    if not button.Visible or not button.Active then
        log("Button not ready")
        return
    end

    log("Auto clicking Phone button")
    clickButton(button)
end

-- Lắng nghe UI xuất hiện
local function watchDeviceSelect()
    local deviceSelect = playerGui:FindFirstChild("DeviceSelect")

    if deviceSelect then
        if deviceSelect.Enabled then
            trySelectPhone(deviceSelect)
        end

        deviceSelect:GetPropertyChangedSignal("Enabled"):Connect(function()
            if deviceSelect.Enabled then
                task.wait(0.1)
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

-- Start
watchDeviceSelect()
