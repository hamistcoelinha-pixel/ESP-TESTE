-- ESP Skeleton + Box Ultra Simples (R6/R15, Menu GUI, FPS MAX)

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")

local showSkeleton = false
local showBox = false
local menuOpen = false
local SKELETON_DIST = 150
local MAX_PLAYERS = 10

-- GUI
local gui = Instance.new("ScreenGui")
gui.Name = "ESP_Menu"
gui.ResetOnSpawn = false
pcall(function() gui.Parent = game:GetService("CoreGui") end)
if not gui.Parent then gui.Parent = LocalPlayer:WaitForChild("PlayerGui") end

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 220, 0, 130)
frame.Position = UDim2.new(0.5, -110, 0.4, 0)
frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
frame.BorderSizePixel = 0
frame.Visible = false
frame.Parent = gui

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 32)
title.BackgroundTransparency = 1
title.Text = "ESP MENU"
title.TextColor3 = Color3.fromRGB(255,255,255)
title.Font = Enum.Font.SourceSansBold
title.TextSize = 20
title.Parent = frame

local skeletonBtn = Instance.new("TextButton")
skeletonBtn.Size = UDim2.new(0.8, 0, 0, 32)
skeletonBtn.Position = UDim2.new(0.1, 0, 0.5, -16)
skeletonBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 120)
skeletonBtn.Text = "ESP Skeleton: OFF"
skeletonBtn.TextColor3 = Color3.fromRGB(255,255,255)
skeletonBtn.Font = Enum.Font.SourceSansBold
skeletonBtn.TextSize = 16
skeletonBtn.Parent = frame

local boxBtn = Instance.new("TextButton")
boxBtn.Size = UDim2.new(0.8, 0, 0, 32)
boxBtn.Position = UDim2.new(0.1, 0, 0.5, 24)
boxBtn.BackgroundColor3 = Color3.fromRGB(40, 120, 40)
boxBtn.Text = "ESP Box: OFF"
boxBtn.TextColor3 = Color3.fromRGB(255,255,255)
boxBtn.Font = Enum.Font.SourceSansBold
boxBtn.TextSize = 16
boxBtn.Parent = frame

UIS.InputBegan:Connect(function(input, processed)
    if not processed and input.KeyCode == Enum.KeyCode.Insert then
        menuOpen = not menuOpen
        frame.Visible = menuOpen
    end
end)
skeletonBtn.MouseButton1Click:Connect(function()
    showSkeleton = not showSkeleton
    skeletonBtn.Text = "ESP Skeleton: " .. (showSkeleton and "ON" or "OFF")
    skeletonBtn.BackgroundColor3 = showSkeleton and Color3.fromRGB(80,80,220) or Color3.fromRGB(40,40,120)
end)
boxBtn.MouseButton1Click:Connect(function()
    showBox = not showBox
    boxBtn.Text = "ESP Box: " .. (showBox and "ON" or "OFF")
    boxBtn.BackgroundColor3 = showBox and Color3.fromRGB(80,220,80) or Color3.fromRGB(40,120,40)
end)

local function getRainbowColor()
    local t = tick() * 0.7
    return Color3.fromHSV((t % 1), 1, 1)
end

local cache = {}
Players.PlayerRemoving:Connect(function(p)
    if cache[p.UserId] then
        for _,obj in ipairs(cache[p.UserId]) do
            if obj and obj.Remove then obj:Remove() end
        end
        cache[p.UserId] = nil
    end
end)

RunService.RenderStepped:Connect(function()
    local validPlayers = {}
    for _,player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            local char = player.Character
            local hrp = char.HumanoidRootPart
            local humanoid = char:FindFirstChildOfClass("Humanoid")
            if humanoid and humanoid.Health > 0 then
                local dist = (hrp.Position - Camera.CFrame.Position).Magnitude
                if dist <= SKELETON_DIST then
                    table.insert(validPlayers, {player=player, dist=dist, char=char, hrp=hrp})
                end
            end
        end
    end
    table.sort(validPlayers, function(a,b) return a.dist < b.dist end)
    while #validPlayers > MAX_PLAYERS do table.remove(validPlayers) end

    local drawnNow = {}
    for _,item in ipairs(validPlayers) do
        drawnNow[item.player.UserId] = true
        local player = item.player
        local char = item.char
        local hrp = item.hrp
        local color = getRainbowColor()
        local pos, onScreen = Camera:WorldToViewportPoint(hrp.Position)
        cache[player.UserId] = cache[player.UserId] or {}
        local c = cache[player.UserId]
        -- SKELETON
        if showSkeleton and onScreen then
            local isR15 = char:FindFirstChild("UpperTorso") and char:FindFirstChild("LowerTorso")
            c.skel = c.skel or {}
            for i=1,3 do
                if not c.skel[i] then
                    c.skel[i] = Drawing.new("Line")
                    c.skel[i].Thickness = 2
                    c.skel[i].Transparency = 1
                end
            end
            if isR15 then
                local head = char:FindFirstChild("Head")
                local utorso = char:FindFirstChild("UpperTorso")
                local ltorso = char:FindFirstChild("LowerTorso")
                local luleg = char:FindFirstChild("LeftUpperLeg")
                local ruleg = char:FindFirstChild("RightUpperLeg")
                -- Head→UpperTorso
                if head and utorso then
                    local pa, va = Camera:WorldToViewportPoint(head.Position)
                    local pb, vb = Camera:WorldToViewportPoint(utorso.Position)
                    if va and vb then
                        c.skel[1].Visible = true
                        c.skel[1].From = Vector2.new(pa.X, pa.Y)
                        c.skel[1].To = Vector2.new(pb.X, pb.Y)
                        c.skel[1].Color = color
                    else c.skel[1].Visible = false end
                else c.skel[1].Visible = false end
                -- UpperTorso→LowerTorso
                if utorso and ltorso then
                    local pa, va = Camera:WorldToViewportPoint(utorso.Position)
                    local pb, vb = Camera:WorldToViewportPoint(ltorso.Position)
                    if va and vb then
                        c.skel[2].Visible = true
                        c.skel[2].From = Vector2.new(pa.X, pa.Y)
                        c.skel[2].To = Vector2.new(pb.X, pb.Y)
                        c.skel[2].Color = color
                    else c.skel[2].Visible = false end
                else c.skel[2].Visible = false end
                -- LowerTorso→LeftUpperLeg (preferencial) ou →RightUpperLeg
                if ltorso and luleg then
                    local pa, va = Camera:WorldToViewportPoint(ltorso.Position)
                    local pb, vb = Camera:WorldToViewportPoint(luleg.Position)
                    if va and vb then
                        c.skel[3].Visible = true
                        c.skel[3].From = Vector2.new(pa.X, pa.Y)
                        c.skel[3].To = Vector2.new(pb.X, pb.Y)
                        c.skel[3].Color = color
                    else c.skel[3].Visible = false end
                elseif ltorso and ruleg then
                    local pa, va = Camera:WorldToViewportPoint(ltorso.Position)
                    local pb, vb = Camera:WorldToViewportPoint(ruleg.Position)
                    if va and vb then
                        c.skel[3].Visible = true
                        c.skel[3].From = Vector2.new(pa.X, pa.Y)
                        c.skel[3].To = Vector2.new(pb.X, pb.Y)
                        c.skel[3].Color = color
                    else c.skel[3].Visible = false end
                else c.skel[3].Visible = false end
            else -- R6
                local head = char:FindFirstChild("Head")
                local torso = char:FindFirstChild("Torso")
                local lleg = char:FindFirstChild("Left Leg")
                local rleg = char:FindFirstChild("Right Leg")
                -- Head→Torso
                if head and torso then
                    local pa, va = Camera:WorldToViewportPoint(head.Position)
                    local pb, vb = Camera:WorldToViewportPoint(torso.Position)
                    if va and vb then
                        c.skel[1].Visible = true
                        c.skel[1].From = Vector2.new(pa.X, pa.Y)
                        c.skel[1].To = Vector2.new(pb.X, pb.Y)
                        c.skel[1].Color = color
                    else c.skel[1].Visible = false end
                else c.skel[1].Visible = false end
                -- Torso→Left Leg
                if torso and lleg then
                    local pa, va = Camera:WorldToViewportPoint(torso.Position)
                    local pb, vb = Camera:WorldToViewportPoint(lleg.Position)
                    if va and vb then
                        c.skel[2].Visible = true
                        c.skel[2].From = Vector2.new(pa.X, pa.Y)
                        c.skel[2].To = Vector2.new(pb.X, pb.Y)
                        c.skel[2].Color = color
                    else c.skel[2].Visible = false end
                else c.skel[2].Visible = false end
                -- Torso→Right Leg
                if torso and rleg then
                    local pa, va = Camera:WorldToViewportPoint(torso.Position)
                    local pb, vb = Camera:WorldToViewportPoint(rleg.Position)
                    if va and vb then
                        c.skel[3].Visible = true
                        c.skel[3].From = Vector2.new(pa.X, pa.Y)
                        c.skel[3].To = Vector2.new(pb.X, pb.Y)
                        c.skel[3].Color = color
                    else c.skel[3].Visible = false end
                else c.skel[3].Visible = false end
            end
        elseif cache[player.UserId].skel then
            for i=1,3 do if cache[player.UserId].skel[i] then cache[player.UserId].skel[i].Visible = false end end
        end
        -- BOX
        if showBox and onScreen then
            if not c.box then
                c.box = Drawing.new("Square")
                c.box.Filled = false
                c.box.Thickness = 2
                c.box.Transparency = 1
            end
            c.box.Visible = true
            c.box.Position = Vector2.new(pos.X-25,pos.Y-50)
            c.box.Size = Vector2.new(50,100)
            c.box.Color = color
        elseif c.box then
            c.box.Visible = false
        end
    end
    -- Esconde/desativa desenhos de jogadores não próximos
    for userId, c in pairs(cache) do
        if not drawnNow[userId] then
            if c.skel then for i=1,3 do if c.skel[i] then c.skel[i].Visible = false end end end
            if c.box then c.box.Visible = false end
        end
    end
end)

print("[ESP SKELETON+BOX SUPER LEVE] Pressione INSERT para abrir o menu e clique nos botões para ativar/desativar!")
