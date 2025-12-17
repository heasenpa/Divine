local Players = game:GetService("Players")
local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

local function log(msg)
    print("[PhoneAutoSelect] " .. msg)
end

-- Keep checking until the Phone.Button appears and is ready
task.spawn(function()
    while true do
        local deviceSelect = playerGui:FindFirstChild("DeviceSelect")
        if deviceSelect and deviceSelect.Enabled then
            local container = deviceSelect:FindFirstChild("Container")
            local phone = container and container:FindFirstChild("Phone")
            local button = phone and phone:FindFirstChild("Button")

            if button and (button:IsA("TextButton") or button:IsA("ImageButton")) then
                if button.Visible then
                    log("Firing Phone button click event.")
                    firesignal(button.MouseButton1Click)
                    break
                else
                    log("Button found but not visible yet.")
                end
            else
                log("Phone.Button not found or not a button.")
            end
        end
        task.wait(0.2)
    end
end)
