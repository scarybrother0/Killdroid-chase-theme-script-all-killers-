# Killdroid-chase-theme-script-all-killers- <--- don't copy this
--[[
	WARNING: Heads up! This script has not been verified by ScriptBlox. Use at your own risk!
]]
local ShouldItBeOnlyOnArtful = false

local SOUND_URL = "https://nu.vgmtreasurechest.com/soundtracks/die-of-death-roblox-2025/gqlfvlvr/23.%20Insubstantial%20%28Killdroid%20Theme%29.mp3"
local LOCAL_FOLDER = "GameSounds"
local LOCAL_FILENAME = "ArtfulTheme.mp3"
local TARGET_SOUND_NAME = "ChaseTheme"
local VOLUME = 4

local ReplicatedStorage = game:GetService("ReplicatedStorage")
local CheckKillerName = ReplicatedStorage:FindFirstChild("CheckKillerName")

if not CheckKillerName then
    CheckKillerName = Instance.new("BoolValue")
    CheckKillerName.Name = "CheckKillerName"
    CheckKillerName.Value = ShouldItBeOnlyOnArtful
    CheckKillerName.Parent = ReplicatedStorage
end

local function fetchUrl(url)
    local attempts = {}

    if syn and syn.request then
        table.insert(attempts, function()
            local ok, res = pcall(function()
                return syn.request({Url=url, Method="GET", Headers={["User-Agent"]="Roblox"}})
            end)
            if ok and res and (res.Body or res.body) then
                return true, (res.Body or res.body)
            end
            return false
        end)
    end

    if http and http.request then
        table.insert(attempts, function()
            local ok,res = pcall(function() return http.request({Url=url, Method="GET"}) end)
            if ok and res and (res.Body or res.body) then return true,(res.Body or res.body) end
            return false
        end)
    end

    if request then
        table.insert(attempts, function()
            local ok,res = pcall(function() return request({Url=url, Method="GET"}) end)
            if ok and res and (res.Body or res.body) then return true,(res.Body or res.body) end
            return false
        end)
    end

    table.insert(attempts, function()
        local ok, body = pcall(function() return game:HttpGet(url,true) end)
        if ok and body then return true,body end
        return false
    end)

    for _,fn in ipairs(attempts) do
        local ok, body = fn()
        if ok and body then return true,body end
    end
    return false
end

local function writeLocalFile(folder, filename, data)
    if not (makefolder and isfolder and writefile and isfile) then return false,"no file API" end
    if not isfolder(folder) then pcall(makefolder, folder) end
    local path = folder.."/"..filename
    local ok,err = pcall(function() writefile(path,data) end)
    if not ok then return false,err end
    if not isfile(path) then return false,"file missing after write" end
    return true,path
end

local function localPathToAssetId(path)
    local funcs = {getcustomasset, getsynasset}
    for _,fn in ipairs(funcs) do
        if type(fn)=="function" then
            local ok,res = pcall(fn,path)
            if ok and type(res)=="string" then return true,res end
        end
    end
    return false
end

local FADE_TIME = 2

local function fadeSound(sound, targetVolume, fadeTime)
    task.spawn(function()
        local startVolume = sound.Volume
        local diff = targetVolume - startVolume
        local stepTime = 0.05
        local steps = math.ceil(fadeTime / stepTime)
        for i = 1, steps do
            sound.Volume = startVolume + diff * (i / steps)
            task.wait(stepTime)
        end
        sound.Volume = targetVolume
    end)
end

local function applyChaseTheme(model, assetId)
    if not model or not model:IsA("Model") then return end
    local anims = model:FindFirstChild("Animations")
    if not anims then return end

    local sound = anims:FindFirstChild(TARGET_SOUND_NAME)
    if sound and sound:IsA("Sound") then
        if sound.IsPlaying then
            fadeSound(sound, 0, FADE_TIME)
            task.wait(FADE_TIME)
        end

        sound.SoundId = assetId
        sound:Play()
        sound.Volume = 0
        fadeSound(sound, VOLUME, FADE_TIME)

        if sound:GetAttribute("CalmTheme") ~= nil then
            sound:SetAttribute("CalmTheme", assetId)
        end
        if sound:GetAttribute("EnragedTheme") ~= nil then
            sound:SetAttribute("EnragedTheme", assetId)
        end

        print(("[Artful-is-tuff] Applied chase theme to %s (Check=%s, Volume=%.2f)"):format(
            model.Name, tostring(CheckKillerName.Value), sound.Volume))
    end
end




do
    print("[Artful-is-tuff] Fetching custom sound...")
    local ok, body = fetchUrl(SOUND_URL)
    if not ok then warn("[Artful-is-tuff] HTTP fetch failed"); return end

    local writeOk, path = writeLocalFile(LOCAL_FOLDER, LOCAL_FILENAME, body)
    if not writeOk then warn("[Artful-is-tuff] Write failed:",path); return end

    local okAsset, assetId = localPathToAssetId(path)
    if not okAsset then warn("[Artful-is-tuff] Failed to map asset id"); return end

    local killerFolder = workspace:WaitForChild("GameAssets"):WaitForChild("Teams"):WaitForChild("Killer")

local function monitorTransitionSounds(model, assetId)
    local anims = model:FindFirstChild("Animations")
    if not anims then return end

    local transitionSound = anims:FindFirstChild("Transition")
    if transitionSound and transitionSound:IsA("Sound") then
        transitionSound.Ended:Connect(function()
            applyChaseTheme(model, assetId)
        end)
    end
end

local function scanAll()
    for _, model in ipairs(killerFolder:GetChildren()) do
        if not CheckKillerName.Value or model:GetAttribute("KillerName") == "Artful" then
            applyChaseTheme(model, assetId)
            monitorTransitionSounds(model, assetId)
        end
    end
end

killerFolder.ChildAdded:Connect(function(model)
    task.wait(0.5)
    if not CheckKillerName.Value or model:GetAttribute("KillerName") == "Artful" then
        applyChaseTheme(model, assetId)
        monitorTransitionSounds(model, assetId)
    end
end)

    local function scanAll()
        for _,model in ipairs(killerFolder:GetChildren()) do
            if not CheckKillerName.Value or model:GetAttribute("KillerName") == "Artful" then
                applyChaseTheme(model, assetId)
            end
        end
    end

    scanAll()

    killerFolder.ChildAdded:Connect(function(model)
        task.wait(0.5)
        if not CheckKillerName.Value or model:GetAttribute("KillerName") == "Artful" then
            applyChaseTheme(model, assetId)
        end
    end)

    CheckKillerName:GetPropertyChangedSignal("Value"):Connect(function()
        print("[Artful-is-tuff] Toggle changed! Reapplying...")
        scanAll()
    end)

    print("[Artful-is-tuff] Ready. Auto-update active.")
end
