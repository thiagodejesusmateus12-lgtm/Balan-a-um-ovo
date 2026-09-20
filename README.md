-- HUD Compacto: Locais + Velocidade + Auto Safe + Treino
local R = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()
local P = game:GetService("Players")
local RS = game:GetService("RunService")
local W = game:GetService("Workspace")
local LP = P.LocalPlayer
local C = {Smin=10,Smax=550,Sdef=100,Amin=10,Amax=10000,Adef=300,Tn="TreadmillSpawn",Ts=200,To=3}
local D = {
    {"Safe",20.8744354,5,134.934174},{"Planície Verde",10.7006378,6.00000095,295.06842},
    {"Floresta Exuberante",9.938591,6.5,598.787292},{"Dunas do Deserto",49.3189926,5.00000095,889.614197},
    {"Praia",8.19842148,5.00000191,1217.01208},{"Terra do Doce",10.5017996,6,1498.20703},
    {"Savana Selvagem",8.96596527,6,1849.20789},{"Vulcão",13.5877132,5.00000095,2238.68652},
    {"Vale Dino",13.1846743,6,2663.6377},{"Oceano Profundo",15.4916439,5.00000095,3094.66626},
}
for _,d in ipairs(D) do d[1],d[2] = d[1],CFrame.new(d[2],d[3],d[4]) end
local SF,SN = D[1][2],D[1][1]
local S = {Sp=C.Sdef,ASp=C.Adef,Ch=nil,Hm=nil,Rp=nil,Fly=false,Dest=nil,Conn=nil,Ft=0,AE=false,AL=nil,LE=false,AAF=false,AT=false,TL=nil,TP=nil,Walk=false,LTC=0}
local et = {"egg","ovo","egg_pet","carrying_egg","held_egg"}
local ea = {"IsCarryingEgg","HasEgg","CarryingEgg","HoldingEgg","EggEquipped","EggSelected"}
local ic = {Tool=true,Backpack=true,HopperBin=true,Accessory=true}

local function RC()
    S.Ch = LP.Character or LP.CharacterAdded:Wait()
    S.Hm = S.Ch:WaitForChild("Humanoid")
    S.Rp = S.Ch:WaitForChild("HumanoidRootPart")
end
LP.CharacterAdded:Connect(function() RC(); S.LE=false end)
if LP.Character then RC() end

local function nameEgg(n)
    if type(n)~="string" then return false end
    n = n:lower()
    for _,t in ipairs(et) do if n:find(t,1,true) then return true end end
    return false
end

local function hasEA(o)
    if not o then return false end
    local ok,a = pcall(function() return o:GetAttributes() end)
    if ok and a then for k,v in pairs(a) do
        for _,e in ipairs(ea) do if k==e and v~=false and v~=0 and v~="" then return true end end
    end end
    for _,c in ipairs(o:GetChildren()) do
        if c:IsA("ValueBase") or c:IsA("ObjectValue") then
            for _,e in ipairs(ea) do if c.Name==e then
                local v = c.Value
                if v~=false and v~=0 and v~="" and v~=nil then return true end
            end end
        end
    end
    return false
end

local function evalO(o)
    if not o then return false end
    if hasEA(o) then return true end
    if o:IsA("Tool") or ic[o.ClassName] then return false end
    if (o:IsA("Model") or o:IsA("BasePart") or o:IsA("Folder") or o:IsA("Configuration")) and nameEgg(o.Name) then return true end
    return false
end

local function isEgg()
    if not S.Ch then return false end
    if hasEA(S.Ch) then return true end
    for _,c in ipairs(S.Ch:GetChildren()) do
        if evalO(c) then return true end
        if c:IsA("Model") or c:IsA("Folder") then
            for _,s in ipairs(c:GetChildren()) do if evalO(s) then return true end end
        end
    end
    return false
end

local function findPlot()
    local n = LP.Name:lower()
    local cands = {}
    for _,o in ipairs(W:GetChildren()) do
        if o~=S.Ch and (o:IsA("Model") or o:IsA("Folder") or o:IsA("BasePart")) then
            local ol = o.Name:lower()
            if (ol:find("plot",1,true) and ol:find(n,1,true)) or ol==n then
                table.insert(cands,o)
            else
                local ok,ow = pcall(function() return o:GetAttribute("Owner") end)
                if ok and type(ow)=="string" and ow:lower()==n then table.insert(cands,o) end
            end
        end
    end
    for _,p in ipairs(cands) do
        local t = p:FindFirstChild(C.Tn,true)
        if t then return p,t end
    end
    return cands[1],nil
end

local function findT()
    local p,t = findPlot()
    if t then return t,p end
    local f = W:FindFirstChild(C.Tn,true)
    if f and f:IsA("BasePart") then return f,f.Parent end
    return nil,p
end

local function cancelF(msg)
    if S.Conn then S.Conn:Disconnect(); S.Conn=nil end
    if S.Hm then pcall(function()
        S.Hm.PlatformStand=false
        S.Hm:ChangeState(Enum.HumanoidStateType.GettingUp)
        S.Hm:ChangeState(Enum.HumanoidStateType.Running)
    end) end
    if S.Rp then pcall(function()
        S.Rp.Anchored=false
        S.Rp.Velocity=Vector3.zero
        S.Rp.RotVelocity=Vector3.zero
    end) end
    S.Fly=false; S.Ft=0
    if msg then R:Notify({Title="Voo Cancelado",Content="Interrompido.",Duration=3}) end
end

local function setSp(v) S.Sp = math.clamp(v,C.Smin,C.Smax) end
local function setASP(v) S.ASp = math.clamp(v,C.Amin,C.Amax) end

local function flyTo(cf,name,auto,onArr,spdOv)
    cancelF(false)
    if not S.Ch or not S.Ch.Parent then RC() end
    if not S.Rp or not S.Rp.Parent then RC() end
    if not S.Hm or S.Hm.Health<=0 then
        if not auto then R:Notify({Title="Erro",Content="Personagem inválido.",Duration=3}) end
        return
    end
    S.Fly=true; S.Dest=name; S.Ft=tick()
    S.Hm.PlatformStand=true; S.Rp.Anchored=true
    local rot = S.Rp.CFrame - S.Rp.CFrame.Position
    local tp = cf.Position
    local spd = spdOv or (auto and S.ASp or S.Sp)
    S.Conn = RS.Heartbeat:Connect(function(dt)
        if not S.Fly then return end
        if not S.Rp or not S.Rp.Parent or not S.Hm or S.Hm.Health<=0 then cancelF(true); return end
        if tick()-S.Ft > 60 then cancelF(true); return end
        local cp = S.Rp.Position
        local dl = tp - cp
        local d = dl.Magnitude
        if d <= 5 then
            S.Rp.CFrame = CFrame.new(tp)*rot
            cancelF(false)
            if onArr then task.spawn(onArr) end
            return
        end
        local st = spd*dt
        if st >= d then st = d end
        S.Rp.CFrame = CFrame.new(cp + (dl/d)*st)*rot
    end)
end

local function goT(onArr)
    if S.Fly then cancelF(false) end
    local t = findT()
    if not t then R:Notify({Title="Auto Treinar",Content="Esteira não encontrada.",Duration=3}); return false end
    S.TP = t
    local tp = t.Position + Vector3.new(0,C.To,0)
    local osp = S.Sp; S.Sp = C.Ts
    flyTo(CFrame.new(tp),"Esteira",false,function()
        S.Sp = osp
        if S.Rp and S.Hm and S.Hm.Health>0 then
            task.wait(.15)
            pcall(function() S.Rp.AssemblyLinearVelocity = Vector3.new(0,-50,0) end)
        end
        if onArr then task.spawn(onArr) end
    end)
    return true
end

local function startAL()
    if S.AL then return end
    S.AL = task.spawn(function()
        while S.AE do
            task.wait(.25)
            if not S.AE then break end
            if not S.Ch or not S.Ch.Parent then
                if LP.Character then S.Ch=LP.Character; S.Hm=S.Ch:FindFirstChildOfClass("Humanoid"); S.Rp=S.Ch:FindFirstChild("HumanoidRootPart") end
            end
            local d = isEgg()
            if d and not S.LE then
                S.LE = true
                if not (S.Fly and S.Dest==SN) then
                    if S.Fly then cancelF(false) end
                    S.Walk = false
                    flyTo(SF,SN,true,function()
                        S.LE = false
                        R:Notify({Title="Auto Safe",Content="Chegou no Safe.",Duration=3})
                    end)
                end
            end
            if not d and S.LE then S.LE = false end
        end
        S.AL = nil
    end)
end

local function startTL()
    if S.TL then return end
    S.TL = task.spawn(function()
        while S.AT do
            task.wait(1)
            if not S.AT then break end
            if S.Fly and S.Dest==SN then continue end
            if not S.Ch or not S.Ch.Parent then
                if LP.Character then S.Ch=LP.Character; S.Hm=S.Ch:FindFirstChildOfClass("Humanoid"); S.Rp=S.Ch:FindFirstChild("HumanoidRootPart") end
                continue
            end
            if S.Walk then continue end
            local t = findT()
            if not t then
                if tick()-S.LTC>3 then
                    S.LTC = tick()
                    R:Notify({Title="Auto Treinar",Content="Esteira não encontrada.",Duration=3})
                end
                continue
            end
            S.TP = t
            if S.Rp then
                local d = (S.Rp.Position - t.Position).Magnitude
                if d > 15 then
                    S.Walk = true
                    goT(function() S.Walk = false end)
                end
            end
        end
        S.TL = nil
    end)
end

local Win = R:CreateWindow({
    Name = "Teleport Locations", LoadingTitle = "Carregando...",
    LoadingSubtitle = "HUD de Voo", Theme = "Default",
    ToggleUIKeybind = "K",
    ConfigurationSaving = {Enabled=false},
    Discord = {Enabled=false}, KeySystem = false,
})

-- Aba Locais
local T1 = Win:CreateTab("Locais")
T1:CreateSection("Destinos")
for _,d in ipairs(D) do
    T1:CreateButton({Name=d[1],Callback=function() flyTo(d[2],d[1],false) end})
end

-- Aba Velocidade
local T2 = Win:CreateTab("Velocidade")
T2:CreateSection("Voo")
T2:CreateSlider({Name="Velocidade do Voo",Range={C.Smin,C.Smax},Increment=5,Suffix="s/s",CurrentValue=C.Sdef,Flag="FS",Callback=setSp})
T2:CreateButton({Name="Parar Voo",Callback=function()
    if S.Fly then cancelF(true) else R:Notify({Title="Sem Voo",Content="Nenhum voo ativo.",Duration=3}) end
end})

-- Aba Auto Safe
local T3 = Win:CreateTab("Auto Safe")
T3:CreateSection("Auto Safe")
T3:CreateToggle({Name="Auto Safe (Ovo → Safe)",CurrentValue=false,Flag="AS",Callback=function(v)
    S.AE = v
    if v then
        S.LE = false
        startAL()
        R:Notify({Title="Auto Safe",Content="Ativado.",Duration=3})
    else
        S.LE = false
        R:Notify({Title="Auto Safe",Content="Desativado.",Duration=3})
    end
end})
T3:CreateSlider({Name="Velocidade do Auto Safe",Range={C.Amin,C.Amax},Increment=50,Suffix="s/s",CurrentValue=C.Adef,Flag="ASS",Callback=setASP})
T3:CreateParagraph({Title="Info",Content="Voa pro Safe ao pegar ovo. Fica sempre ativo."})

-- Aba Treino
local T4 = Win:CreateTab("Treino")
T4:CreateSection("Auto Treinar")
T4:CreateToggle({Name="Auto Treinar",CurrentValue=false,Flag="AT",Callback=function(v)
    S.AT = v
    if v then
        S.LTC = 0
        startTL()
        R:Notify({Title="Auto Treinar",Content="Ativado.",Duration=3})
    else
        S.Walk = false
        if S.Fly and S.Dest=="Esteira" then cancelF(false) end
        R:Notify({Title="Auto Treinar",Content="Desativado.",Duration=3})
    end
end})
T4:CreateButton({Name="Ir para a Esteira",Callback=function()
    if not findT() then R:Notify({Title="Auto Treinar",Content="Esteira não encontrada.",Duration=3}); return end
    S.Walk = true
    goT(function() S.Walk = false end)
end})
T4:CreateButton({Name="Parar Treino",Callback=function()
    if S.Fly then cancelF(false) end
    S.Walk = false
    R:Notify({Title="Auto Treinar",Content="Treino parado.",Duration=3})
end})
T4:CreateSection("Status")
T4:CreateButton({Name="Verificar Status",Callback=function()
    local t,p = findT()
    local pn = p and p.Name or "não encontrado"
    local ts = t and "OK" or "NÃO encontrada"
    print("[Status] Plot: "..pn.." | Esteira: "..ts.." | Treinar: "..tostring(S.AT))
    R:Notify({Title="Status",Content="Plot: "..pn.." | Esteira: "..ts,Duration=5})
end})

_G.TL = {flyTo=flyTo, cancelF=cancelF, setSp=setSp, setASP=setASP, findT=findT}
print("[HUD] Carregado. Tecla K para abrir/fechar.")
