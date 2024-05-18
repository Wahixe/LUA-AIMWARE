local player_name = cheat.GetUserName()
local font = draw.CreateFont("Bahnschrift", 35, 100)
local yaw, preva = 0, EulerAngles(nil, nil, nil)

local reference = gui.Reference("Ragebot")
local tab = gui.Tab(reference, "customjitter", "AlphaAA.lua")

local aa_yaw = gui.Groupbox(tab, "Anti-aim Base", 10, 10, 200, 1)
local aa_pitch = gui.Groupbox(tab, "Anti-aim Pitch", 220, 10, 200, 1)

local box_yaw = gui.Combobox(aa_yaw, "box_yaw", "Type Yaw", "Disable", "Jitter", "Random")
local box_pitch = gui.Combobox(aa_pitch, "box_pitch", "Type Pitch", "Disable", "Jitter", "Random")

local speed_yaw = gui.Slider(aa_yaw, "speed_yaw", "Speed", 4, 2, 16, 2)
local jitter1_yaw = gui.Slider(aa_yaw, "jitter1_yaw", "Jitter yaw 1", 10, -180, 180, 1)
local jitter2_yaw = gui.Slider(aa_yaw, "jitter2_yaw", "Jitter yaw 2", 20, -180, 180, 1)

local speed_pitch = gui.Slider(aa_pitch, "speed_pitch", "Speed", 4, 2, 16, 2)
local jitter1_pitch = gui.Slider(aa_pitch, "jitter1_pitch", "Jitter pitch 1", 10, -89, 89, 1)
local jitter2_pitch = gui.Slider(aa_pitch, "jitter2_pitch", "Jitter pitch 2", 20, -89, 89, 1)

local custom_settings = gui.Groupbox(tab, "Custom Settings", 640, 10, 200, 1)
local custom_slider1 = gui.Slider(custom_settings, "custom_slider1", "Custom Slider 1", 0, 0, 100)
local custom_checkbox1 = gui.Checkbox(custom_settings, "custom_checkbox1", "Custom Checkbox 1", false)
local custom_colorpicker1 = gui.ColorPicker(custom_settings, "custom_colorpicker1", "Custom Color Picker 1", 255, 255, 255, 255)

local function checker_func()
    local box_yaw_value = box_yaw:GetValue()
    local box_pitch_value = box_pitch:GetValue()

    speed_yaw:SetInvisible(box_yaw_value == 2)
    speed_yaw:SetDisabled(box_yaw_value == 0)

    speed_pitch:SetInvisible(box_pitch_value == 2)
    speed_pitch:SetDisabled(box_pitch_value == 0)

    jitter1_yaw:SetDisabled(box_yaw_value == 0)
    jitter2_yaw:SetDisabled(box_yaw_value == 0)

    jitter1_pitch:SetDisabled(box_pitch_value == 0)
    jitter2_pitch:SetDisabled(box_pitch_value == 0)
end

local function preva_func(cmd)
    preva = cmd:GetViewAngles()
end

local function antiaim_yaw()
    if box_yaw:GetValue() == 0 then return end

    local tickcount_yaw = globals.TickCount() % speed_yaw:GetValue()
    local fake_yaw = math.sin(tickcount_yaw * 0.1) * 90

    if box_yaw:GetValue() == 1 then
        yaw = tickcount_yaw % 2 == 0 and jitter1_yaw:GetValue() or jitter2_yaw:GetValue() + fake_yaw
    elseif box_yaw:GetValue() == 2 then
        local angle = (tickcount_yaw / speed_yaw:GetValue()) * 360
        yaw = angle + jitter1_yaw:GetValue() + math.sin(math.rad(angle)) * jitter2_yaw:GetValue() + fake_yaw
    end
    gui.SetValue("rbot.antiaim.base", yaw .. " Backward")
end

local function antiaim_pitch(cmd)
    if box_pitch:GetValue() == 0 then return end

    local tickcount_pitch = globals.TickCount() % speed_pitch:GetValue()
    local va = cmd:GetViewAngles()
    local fake_pitch = math.cos(tickcount_pitch * 0.1) * 45

    if va.x == preva.x and va.y == preva.y then return end

    if box_pitch:GetValue() == 1 then
        va.x = tickcount_pitch % 2 == 0 and jitter1_pitch:GetValue() or jitter2_pitch:GetValue() + fake_pitch
    elseif box_pitch:GetValue() == 2 then
        local angle = (tickcount_pitch / speed_pitch:GetValue()) * 360
        va.x = angle + jitter1_pitch:GetValue() + math.sin(math.rad(angle)) * jitter2_pitch:GetValue() + fake_pitch
    end
    cmd:SetViewAngles(va)
end

callbacks.Register("Draw", checker_func)
callbacks.Register("CreateMove", function(cmd)
    antiaim_yaw()
    antiaim_pitch(cmd)
end)
callbacks.Register("PreMove", preva_func)
callbacks.Register("Unload", function()
    gui.SetValue("rbot.antiaim.base", "0 Off")
end)

