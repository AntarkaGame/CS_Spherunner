# CS_Spherunner
CraftStudio Spherunner Project

<p align="center">
    <img src="https://m.gjcdn.net/game-header/2000/25487-qua4h8xd-v4.jpg">
</p>

Site web du jeu: http://spherunner.antarka.com/

Video du jeu: https://www.youtube.com/watch?v=dCa0HUAYXMw

## Requirements
- [CraftStudio](https://sparklinlabs.itch.io/craftstudio)

## Getting Started
Clone the repository at: **AppData\Roaming\CraftStudio\CraftStudioServer\Projects**.
```
$ git clone https://github.com/AntarkaGame/CS_Spherunner.git aa3e4bd0-2a3c-48b6-927c-0fc6b4abbc30
```

> ⚠️ Please be sure to keep the uuid.

## Team

- [fraxken](https://github.com/fraxken) - GENTILHOMME Thomas &lt;gentilhomme.thomas@gmail.com&gt;
- [Aleksander](https://github.com/AlexandreMalaj) - MALAJ Alexandre &lt;alexandre.malaj@gmail.com&gt;
- [Captain](https://github.com/Captainfive) - MONTES Irvin &lt;vinou_montes@hotmail.fr&gt;
- [Chlorodatafile](https://github.com/chlorodatafile) - CARON Gwenaël &lt;gwenael.caron@outlook.com&gt;

## Some code (memory)

<details><summary>Fraxken (bootscreen)</summary>
<br />

```lua
-- > Variable global
hud = nil 
BootScreen = nil 
playlist,playlist_count = nil, 4
Change_text = nil 
ToAction = nil
SkipIntro = nil 
Ingame = false
Movement = nil
camera_statuts = nil
Exited = false
Toclick = true
soundclick = nil
Level_hover = nil 
Intro_stats = nil
Banniere_open = nil
Instance_musique = nil

--> Timer pour la camera
--> Line ~= 300
timerEchap = nil

math.randomseed(os.time())
math.random(); math.random(); math.random()

-- > Awake
function Behavior:Awake()
    self.Allow = true
    self.temps = 0
    self.opa_mode = nil
    self.opa_temps = 0
    
    self.intro_temps = 0
    self.intro_delay = 30
    
    Change_text = false
    BootScreen = self.gameObject
    self.opa_model = CS.FindGameObject("Opa_background").modelRenderer
    self.opa_model:SetOpacity(1.0)
    
    
    -- > Tableau HUD
    hud = {
        active_module = "Accueil",
        object = {
            ["Accueil"] = {
                ["Chunk"] = false,
                ["GameObject"] = CS.FindGameObject("CC_Accueil"),
                ["Pos"] = 300,
                ["Escape"] = "Echap"
            },
            ["Option"] = {
                ["Chunk"] = false,
                ["GameObject"] = CS.FindGameObject("CC_Option"),
                ["Pos"] = 300,
                ["Escape"] = "Accueil"
            },
            ["Solo"] = {
                ["Chunk"] = false,
                ["GameObject"] = CS.FindGameObject("CC_Solo"),
                ["Pos"] = 300,
                ["Escape"] = "Accueil"
            },
            ["Intro"] = {
                ["Chunk"] = false,
                ["GameObject"] = CS.FindGameObject("CC_Intro"),
                ["Pos"] = 300,
                ["Escape"] = "Skip"
            },
            ["End"] = {
                ["Chunk"] = false,
                ["GameObject"] = CS.FindGameObject("CC_End"),
                ["Pos"] = 300,
                ["Escape"] = "Skip"
            }
        }
    }
    
    -- > Playlist
    playlist = {
        [1] = CS.FindAsset( "musique/Menutrack" ),
        [2] = CS.FindAsset( "musique/Maya" ),
        [3] = CS.FindAsset( "musique/track01" ),
        [5] = CS.FindAsset( "musique/track02" ),
        [6] = CS.FindAsset( "musique/track03" ),
        [7] = CS.FindAsset( "musique/CyberTheme" )
    }
    
    
    -- > Lançement du bootscreen et introduction
    if self.Introduction then
        self.introduction = CS.Instantiate( "Key_intro" , CS.FindAsset( "Hud/Introduction" , "Scene" ) )
        self.introduction_start = true
        Intro_stats = true
    else
        self.opa_mode = true
        self.opa_temps = 0
    end
    self.IntroStart = false
    
    if self.ID ~= "" then
        self.showlevel = CS.Instantiate( "Key_showlevel_hud", CS.FindAsset( "Show/"..ID_level, "Scene" ) ) 
        self:Switch("Intro")
        self.IntroStart = true
        
        self.Eau_asset = CS.FindAsset("sound/eau")
        self.Instance_eau = self.Eau_asset:CreateInstance()
        self.Instance_eau:SetLoop( true )
        self.Instance_eau:Play() 
        
        self:Son("water")
    else
        self.level = CS.Instantiate( "Key_level", CS.FindAsset( "Hud/Home_level_alpha" , "Scene" ) ) 
        self.showlevel = CS.Instantiate( "Key_showlevel_hud", CS.FindAsset( "Show/12" , "Scene" ) ) 
        self:Switch("Accueil")
        
        if Instance_musique == nil then
            self.RN = math.random(#playlist)
            Instance_musique = playlist[1]:CreateInstance()
            Instance_musique:SetLoop( false )
            Instance_musique:Play() 
        end
        
        self.Eau_asset = CS.FindAsset("sound/eau")
        self.Instance_eau = self.Eau_asset:CreateInstance()
        self.Instance_eau:SetLoop( true )
        self.Instance_eau:Play() 
        
        self:Son()
        
        -- > Vérifier le premier lancement
        if GameOption.First_start then
            local opa_black = CS.FindGameObject("Opa_black")
            opa_black.modelRenderer:SetOpacity(0.8)
            self.banniere = CS.Instantiate("banniere_key", CS.FindAsset("Hud/Banniere","Scene") )
            Banniere_open = true 
            
            GameOption.First_start = false
            StorageObject:SendMessage("Save",{Message="save"})
        else
            local opa_black = CS.FindGameObject("Opa_black")
            opa_black.modelRenderer:SetOpacity(0)
        end
    end
end

function Behavior:Son(param)
    if param == "water" then
        self.Instance_eau:SetVolume( 0.5 * GameOption.Son.General * GameOption.Son.Environnement )
    else
        Instance_musique:SetVolume( 1 * GameOption.Son.General * GameOption.Son.Musique )
        self.Instance_eau:SetVolume( 0.5 * GameOption.Son.General * GameOption.Son.Environnement )
    end
end

function Behavior:Playlist(item)
    if item[1] == "mute" then
        if item[2] then
            Instance_musique:Pause()
        else
            Instance_musique:Resume()
        end
    else
        if Instance_musique ~= nil then
            if Instance_musique:GetState() == SoundInstance.State.Stopped then
                if playlist_count >= #playlist then
                    playlist_count = 1
                    Instance_musique = playlist[playlist_count]:CreateInstance()
                    Instance_musique:SetLoop( false )
                    self:Son()
                    Instance_musique:Play( )
                else
                    playlist_count = playlist_count + 1
                    Instance_musique = playlist[playlist_count]:CreateInstance()
                    Instance_musique:SetLoop( false )
                    self:Son()
                    Instance_musique:Play( )
                end
            end
        else
            --[[
            Instance_musique = playlist[4]:CreateInstance()
            Instance_musique:SetLoop( false )
            self:Son()
            Instance_musique:Play() --]]
        end
    end
end

function Behavior:Switch(item) 
    if item == "Lobby" then 
        self:Switch( "Solo" )
        CS.LoadScene( CS.FindAsset( "Hud/BootScreen_alpha" , "Scene" ) ) 
        camera_statuts = false
        Movement = false
        Niveau_started = false
        Ingame = false
        CS.Input.UnlockMouse()
        --Instance_musique:Stop()
        self.Instance_eau:Stop()
    elseif item == "Lobby_Home" then
        self:Switch( "Solo" )
        self.opa_mode = false
        ToAction = "Hud/BootScreen_alpha"
        camera_statuts = false
        Movement = false
        Niveau_started = false
        Ingame = false
        CS.Input.UnlockMouse()
        --Instance_musique:Stop()
        self.Instance_eau:Stop()
    elseif item == "End" then
        hud.object[hud.active_module]["GameObject"].transform:SetLocalPosition( Vector3:New( hud.object[hud.active_module]["Pos"] , 0 , -3 ) ) 
        hud.object[hud.active_module]["Chunk"] = false
        hud.object[item]["GameObject"].transform:SetLocalPosition( Vector3:New( 0 , 0 , - 1 ) ) 
        hud.active_module = item 
        hud.object[hud.active_module]["Chunk"] = true
        self:Resize(hud.active_module)
    else
        -- > Switch 
        if self.ID ~= "" and item == "Solo" then
            Movement = true
            --camera_statuts = true
            timerEchap = 0
            CS.Input.LockMouse()
            Exited = false
        end
        
        hud.object[hud.active_module]["GameObject"].transform:SetLocalPosition( Vector3:New( hud.object[hud.active_module]["Pos"] , 0 , -3 ) ) 
        hud.object[hud.active_module]["Chunk"] = false
        hud.object[item]["GameObject"].transform:SetLocalPosition( Vector3:New( 0 , 0 , - 3 ) ) 
        hud.active_module = item 
        hud.object[hud.active_module]["Chunk"] = true
        self:Resize(hud.active_module)
    end
end

function Behavior:Resize(item)  
    if Ingame == true then
        self.Timer_obj = CS.FindGameObject("Hud_time")
        self.Timer_obj.transform:SetLocalPosition( Vector3:New( -4 , SGY*PixelToUnit / 2 - 2 , - 2 ) ) 
    else
        local social = CS.FindGameObject("Social")
        social.transform:SetLocalPosition( Vector3:New( -SGX*PixelToUnit / 2 + 0.5, -SGY*PixelToUnit / 2 + 0.5 , - 3 ) ) 
    end
    local x1 = CS.FindGameObject(item.."_Menu")
    local x2 = CS.FindGameObject(item.."_Container")
    local GameVersion = CS.FindGameObject("GameVersion")
    local sound_mute = CS.FindGameObject("Sound")
    x1.transform:SetLocalPosition( Vector3:New( -SGX*PixelToUnit / 2 - 0.1 ,0, -3 ) ) 
    if hud.object["Accueil"]["Chunk"] ~= true then
        x2.transform:SetLocalPosition( Vector3:New( -SGX*PixelToUnit / 2 + 33 , 0 , - 3 ) )
    end
    GameVersion.transform:SetLocalPosition( Vector3:New( SGX*PixelToUnit / 2 - 5 , -SGY*PixelToUnit / 2 + 0.8 , - 3 ) )
    sound_mute.transform:SetLocalPosition( Vector3:New( SGX*PixelToUnit / 2 - 0.5, SGY*PixelToUnit / 2 - 0.5 , - 3 ) ) 
end

function Behavior:Data(data)
    if data.Message == "Switch" and self.Allow == true then
        self.Allow = false
        self:Switch(data.Value) 
    elseif data.Message == "intro" then
        CS.Destroy( self.introduction )
        self.introduction_start = false
        Intro_stats = false
        self.opa_mode = true
    elseif data.Message == "Son" then
        self:Son()
    elseif data.Message == "mute" then
        self:Playlist({"mute",data.Value})
    elseif data.Message == "StartLevel" then
        self.opa_mode = false
    elseif data.Message == "banniere" then
        CS.Destroy( self.banniere ) 
        local opa_black = CS.FindGameObject("Opa_black")
        opa_black.modelRenderer:SetOpacity(0)
        Banniere_open = nil
    elseif data.Message == "Fade" then
        self.opa_mode = data.Value
    elseif data.Message == "NextLevel" then
        self:Switch("Solo")
        Niveau_started = false
        Movement = false
        camera_statuts = false
        Ingame = false
        NUMBER_level = tonumber(NUMBER_level) + 1 
        if tonumber(NUMBER_level) >= 10 then
            CS.LoadScene( CS.FindAsset( "Level/"..tostring(NUMBER_level), "Scene" ) ) 
        else
            CS.LoadScene( CS.FindAsset( "Level/0"..tostring(NUMBER_level), "Scene" ) ) 
        end
        CS.Input.UnlockMouse()
        --Instance_musique:Stop()
        self.Instance_eau:Stop() 
    elseif data.Message == "Restart" then
        Movement = true
        camera_statuts = true
        Respawn = true
        CS.Input.LockMouse()
        Level_time = nil
        self.opa_mode = true
        self:Switch( "Solo" )
        self.End_obj = CS.FindGameObject("End")
        self.End_obj:SendMessage( "Data", {Message="Active"} ) 
        self.Start_obj = CS.FindGameObject("Start")
        self.Start_obj:SendMessage( "Data", { Message="Respawn" } )
        InWater = true
    end
end


function Behavior:StartLevel()
    self.IntroStart = false
    Ingame = true
    CS.Destroy( self.showlevel ) 
    self.start_level_point = CS.FindGameObject("Start")
    self.start_level_point:SendMessage("Data",{Message="CreateSphere"})
    self:Switch("Solo")
    self.opa_mode = true
    SkipIntro = false
    self.intro_temps = 0
end

function Behavior:Update()

    self.Allow = true
    self.temps = self.temps + 1 / 60 
    self:Playlist({"other",nil})
    
    -- > Gestion de la transition d'opacité
    if self.opa_mode == true then
        self.opa_temps = self.opa_temps + 1 
        self.opa_model:SetOpacity( self.opa_model:GetOpacity() - (1 / 60) ) 
        if self.opa_temps >= 60 then
            self.opa_model:SetOpacity(0)
            self.opa_mode = nil
            self.opa_temps = 0
            SkipIntro = nil 
        end
    end
    
    if self.opa_mode == false then
        self.opa_temps = self.opa_temps + 1 
        self.opa_model:SetOpacity( self.opa_model:GetOpacity() + (1 / 60) ) 
        if self.opa_temps >= 60 then
            self.opa_model:SetOpacity(1.0)
            self.opa_mode = nil
            if ToAction ~= nil then
                --Instance_musique:Stop()
                self.Instance_eau:Stop() 
                CS.LoadScene( CS.FindAsset( ToAction , "Scene" ) ) 
                Toclick = true
                ToAction = nil 
            end
            if SkipIntro == true then
                -- > Lancement du niveau 
                self:StartLevel()
            end
            self.opa_temps = 0
        end
    end
    
    -- > Escape master ! 
    if CS.Input.WasButtonJustPressed( "Echap" ) and SkipIntro == nil then
        if self.introduction_start ~= true then 
            if hud.object["End"]["Chunk"] == true then 
                --print("nothing")
            else    
                if hud.object[hud.active_module]["Escape"] == "Echap" then
                   if self.ID ~= "" then
                       Movement = true
                       camera_statuts = true
                       CS.Input.LockMouse()
                       self:Switch("Solo")
                       Exited = false
                   else
                       StorageObject:SendMessage("Data", {Message="save"} ) 
                       CS.Exit()
                   end
                elseif hud.object[hud.active_module]["Escape"] == "Skip" and SkipIntro == nil then
                    self.opa_mode = false
                    SkipIntro = true
                else
                    if self.ID ~= "" then
                        if timerEchap == nil then
                            timerEchap = 0
                            if hud.object[hud.active_module]["Escape"] == "Accueil" then
                                self:Switch(hud.object[hud.active_module]["Escape"]) 
                                -- > menu du jeu
                                Movement = false
                                camera_statuts = false
                                CS.Input.UnlockMouse()
                                Exited = true
                            else
                                self:Switch(hud.object[hud.active_module]["Escape"]) 
                                -- > retour au jeu
                            end
                        end
                    else
                        self:Switch(hud.object[hud.active_module]["Escape"]) 
                    end
                end
            end
        end
    end
    
    -- > Resizing Master
    if Other_resizing == true then
        self:Resize(hud.active_module)
        Other_resizing = false
    end
    
    -- > AutoSave
    if self.temps >= 5 then
        StorageObject:SendMessage("Data", {Message="save"} ) 
        if Change_text == true then
            Change_text = false
        end
        self.temps = 0
    end
    
    if self.IntroStart == true then
        self.intro_temps = self.intro_temps + 1 / 60
        if self.intro_temps >= self.intro_delay then
            self:StartLevel()
        end
    end
    
end
```
</details>

<details><summary>Aleksander (Camera Behavior)</summary>
<br />

```lua
camera_statuts = nil 
cameraAngleTeta = nil
cameraAnglePhi = nil
ActuGameCam = nil

function Behavior:Awake()
    
    
    self.TreeContainer = CS.FindGameObject("Tree_container")
    if self.TreeContainer ~= nil then
        self.tabTreeObj = self.TreeContainer:GetChildren()
    else
        self.tabTreeObj = nil
    end
    
    
    self.tabDPC = GameOption.Scroll_list
    self.increDPC = GameOption.Scroll
    
    camera_statuts = true
    
    self.SPEED_ROTATE = 0.002
    CS.Input.LockMouse()
    
    self.Ball = CS.FindGameObject("Sphere")
    self.BallPos = self.Ball.transform:GetPosition()
    self.CamPos  = self.gameObject.transform:GetPosition()
    
    self.BallPosOffset = Vector3:New(self.BallPos.x , self.BallPos.y + 4, self.BallPos.z + 7)
    self.gameObject.transform:SetPosition(self.BallPosOffset)
    
    self.DPC    = self.tabDPC[self.increDPC]
    cameraAngleTeta   = math.rad(self.startAngles)
    cameraAnglePhi    = math.rad(30)
    
    self.StartAngleTeta = cameraAngleTeta
    self.StartAnglePhi  = cameraAnglePhi
    
    self.timer = 0
    self.scroll = false
    
    self.w = 0
    self.timerEchap = nil
    
    ActuGameCam = self.gameObject
end

function Behavior:Data(data) 
    if data.Message == "rs" then 
        self.MouseDelta = CS.Input.GetMouseDelta()
        
        if CheckPoint == false then            
            cameraAngleTeta       = self.StartAngleTeta - self.MouseDelta.x * self.SPEED_ROTATE
        else
            cameraAngleTeta       = math.rad(Checkpoint_angle) - self.MouseDelta.x * self.SPEED_ROTATE
        end
        cameraAnglePhi        = self.StartAnglePhi + self.MouseDelta.y * self.SPEED_ROTATE 
          
        -- Gestion des axes souris.
        cameraAngleTeta = cameraAngleTeta % (2*math.pi)
        
        if cameraAnglePhi > math.pi/2.5 then
            cameraAnglePhi = math.pi/2.5;
        elseif cameraAnglePhi < 0 then
            cameraAnglePhi = 0
        end
        self.w = 0
            
        self.BallPos = self.Ball.transform:GetPosition()
        self.NewCamPos  = self.BallPos + self:NewPosCam(cameraAngleTeta,cameraAnglePhi,self.ScrollInter)
        self.gameObject.transform:SetPosition(self.NewCamPos)
    end
end

function Behavior:Update()
    self.BallPos = self.Ball.transform:GetPosition()
    if camera_statuts then
        
        
        if timerEchap == nil then
        
            self.MouseDelta = CS.Input.GetMouseDelta()
                    
            cameraAngleTeta       = cameraAngleTeta - self.MouseDelta.x * self.SPEED_ROTATE
            cameraAnglePhi        = cameraAnglePhi + self.MouseDelta.y * self.SPEED_ROTATE 
              
            -- Gestion des axes souris.
            cameraAngleTeta = cameraAngleTeta % (2*math.pi)
            
            if cameraAnglePhi > math.pi/2.5 then
                cameraAnglePhi = math.pi/2.5;
            elseif cameraAnglePhi < 0 then
                cameraAnglePhi = 0
            end
            self.w = 0
            
            if self.tabTreeObj ~= nil then
                local BallPos = self.Ball.transform:GetPosition()
                self.NewRay = Ray:New( BallPos ,Vector3.Rotate( Vector3:New( 0, 0, 1 ), self.gameObject.transform:GetOrientation() ) )
                local distance = self:InterModelRenderer( self.NewRay, self.tabTreeObj )
                
                if distance ~= nil and distance < self.DPC then
                    self.ScrollInter = distance - 0.6
                else
                    self.ScrollInter = self.DPC
                end
            else
                self.ScrollInter = self.DPC
            end
            
            self.NewCamPos  = self.BallPos + self:NewPosCam(cameraAngleTeta,cameraAnglePhi,self.ScrollInter)
            self.gameObject.transform:SetPosition(self.NewCamPos);    
        
            if self.scroll then
                if self.timer ~= nil then
                    self.timer = self.timer + 1
                    local factor = self.timer / 60
                    self.CamPosStart = self.gameObject.transform:GetPosition()
                    self.CamPosEnd = self.BallPos + self:NewPosCam(cameraAngleTeta,cameraAnglePhi,self.tabDPC[self.increDPC])
                    local VectorSmoothStep = Smoothstep(self.CamPosStart , self.CamPosEnd, factor)
                    self.gameObject.transform:SetPosition(VectorSmoothStep)
                
                    if self.timer > 60 then
                        self.timer = nil
                        self.DPC = self.tabDPC[self.increDPC]
                        self.scroll = false
                    end
                end
            else
                -- Gestion du scroll UP et DOWN (Molette souris).
                if CS.Input.IsButtonDown("Scroll_down") then
                    self.scroll = true
                    self.timer = 0
                    self.increDPC = self.increDPC - 1
                    if self.increDPC < 1 then
                        self.increDPC = 1
                        GameOption.Scroll = self.increDPC
                        StorageObject:SendMessage("Save",{Message="save"})
                        self.scroll = false
                        self.timer = nil
                    end
                elseif CS.Input.IsButtonDown("Scroll_up") then
                    self.scroll = true
                    self.timer = 0
                    self.increDPC = self.increDPC + 1
                    if self.increDPC > #self.tabDPC then
                        self.increDPC = #self.tabDPC
                        GameOption.Scroll = self.increDPC
                        StorageObject:SendMessage("Save",{Message="save"})
                        self.scroll = false
                        self.timer = nil
                    end
                end
            end

        else
            timerEchap = timerEchap + 1
            print(timerEchap)
            local factor = timerEchap / 60
            self.radW = math.rad(self.w)
            self.CamPosStart = self.BallPos + self.BallPos + Vector3:New(    10 * math.sin( self.radW ),
                                                            self.BallPos.y + 4,
                                                            10 * math.cos( self.radW ))
            self.CamPosEnd = self.BallPos + self:NewPosCam(cameraAngleTeta,cameraAnglePhi,self.tabDPC[self.increDPC])
            local VectorSmoothStep = Smoothstep(self.CamPosStart, self.CamPosEnd, factor)
            self.gameObject.transform:SetPosition(VectorSmoothStep)
            if timerEchap >= 60 then
                timerEchap = nil
                self.w = 0
            end
        end
        
    else 
         if timerEchap == nil then
            self.w = self.w + 1
            if self.w >= 360 then
                self.w = 0
            end
            self.radW = math.rad(self.w)
            self.CamPosEnd = self.BallPos + Vector3:New(    10 * math.sin( self.radW ),
                                                            self.BallPos.y + 4,
                                                            10 * math.cos( self.radW ))
            self.gameObject.transform:SetPosition(self.CamPosEnd)
        else
            timerEchap = timerEchap + 1
            local factor = timerEchap / 60
            self.CamPosEnd = self.BallPos + Vector3:New( 10* math.sin(0), self.BallPos.y + 4, 10 * math.cos(0) )
            self.CamPosStart = self.BallPos + self:NewPosCam(cameraAngleTeta,cameraAnglePhi,self.tabDPC[self.increDPC])
            --self.CamPosEnd =  self:NewPosCam(0,0,10)
            
            local VectorSmoothStep = Smoothstep(self.CamPosStart , self.CamPosEnd, factor)
            self.gameObject.transform:SetPosition(VectorSmoothStep)
            if timerEchap >= 60 and Exited then
                timerEchap = nil
            elseif timerEchap >= 60 and not Exited then
                timerEchap = nil
                camera_statuts = true
            end
        end
    end
    self.gameObject.transform:LookAt(self.BallPos)
end



function Behavior:NewPosCam(teta,phi,Scroll)
    Scroll = Scroll or 0
    local NewPos = Vector3:New( 
        Scroll * math.sin( teta ) * math.cos( phi ),
        Scroll * math.sin( phi ),
        Scroll * math.cos( teta ) * math.cos( phi )
    )
    return NewPos
end

function Smoothstep( a, b, v )
    local v = 1 - (1 - v) * (1 - v)
    local out = Vector3:New(0)
    out.x = (b.x * v) + (a.x *(1 - v))
    out.y = (b.y * v) + (a.y *(1 - v))
    out.z = (b.z * v) + (a.z *(1 - v))
    return out
end

function Behavior:InterModelRenderer(ray,tab)
    for i = 1,#tab,1 do
        local distance = ray:IntersectsModelRenderer( tab[i]:GetComponent("ModelRenderer") )
        if distance ~= nil then
            return distance, tab[i]
        end
    end
    return nil
end

```
</details>

<details><summary>Chlorodatafile (Box that fall in water)</summary>
<br />

```lua
function Behavior:Awake()

    self.Water      = CS.FindGameObject("WaterRenderer")
    self.WaterPosY   = self.Water.transform:GetPosition().y
    self.BoxPosX     = self.gameObject.transform:GetPosition().x
    self.BoxPosZ     = self.gameObject.transform:GetPosition().z
    self.gameObject.transform:SetPosition(
        Vector3:New(
            self.BoxPosX,
            self.WaterPosY,
            self.BoxPosZ
        )
    )
    self.NewAngle = self.gameObject.transform:GetEulerAngles()
    
    self.AngleX = math.randomrange( self.AngleXMin, self.AngleXMax )
    self.SaveAngleX = self.AngleX
    
    self.AngleY = math.randomrange( self.AngleYMin, self.AngleYMax )
    self.SaveAngleY = self.AngleY
    
    self.AngleZ = math.randomrange( self.AngleZMin, self.AngleZMax )
    self.SaveAngleZ = self.AngleZ
    
    self.DurationTimerX = math.randomrange( self.RandomTimeMin, self.RandomTimeXMax )
    self.DurationTimerY = math.randomrange( self.RandomTimeMin, self.RandomTimeYMax )
    self.DurationTimerZ = math.randomrange( self.RandomTimeMin, self.RandomTimeZMax )
    
    self.timerX = nil
    self.timerY = nil
    self.timerZ = nil
    
    
end

function Behavior:Update()

    self.WaterPosY = self.Water.transform:GetPosition().y
    self.gameObject.physics:WarpPosition(
        Vector3:New(
            self.BoxPosX,
            self.WaterPosY,
            self.BoxPosZ
        )
    )
    
    --> ANGLE X
    if self.timerX ~= nil then
        local factor   = self.timerX / self.DurationTimerX
        self.NewAngle.x = self:SmoothstepX( self.StartAngleX, self.EndAngleX, factor )
        
        if self.timerX >= self.DurationTimerX then
            self.timerX = nil
        else
            self.timerX = self.timerX + 1
        end
    else
        self.AngleX = math.randomrange( self.AngleXMin - self.SaveAngleX, self.AngleXMax - self.SaveAngleX )
        self.SaveAngleX = self.SaveAngleX + self.AngleX
        self.timerX = 0
        
        self.StartAngleX = self.gameObject.transform:GetEulerAngles().x
        self.EndAngleX   = self.StartAngleX + self.AngleX
    end
    
    --> ANGLE Y
    if self.timerY ~= nil then
        local factor   = self.timerY / self.DurationTimerY
        self.NewAngle.y = self:SmoothstepY( self.StartAngleY, self.EndAngleY, factor )
        
        if self.timerY >= self.DurationTimerY then
            self.timerY = nil
        else
            self.timerY = self.timerY + 1
        end
    else
        self.AngleY = math.randomrange( self.AngleYMin - self.SaveAngleY, self.AngleYMax - self.SaveAngleY )
        self.SaveAngleY = self.SaveAngleY + self.AngleY
        self.timerY = 0
        
        self.StartAngleY = self.gameObject.transform:GetEulerAngles().y
        self.EndAngleY   = self.StartAngleY + self.AngleY
    end
    
    --> ANGLE Z
    if self.timerZ ~= nil then
        local factor   = self.timerZ / self.DurationTimerZ
        self.NewAngle.z = self:SmoothstepZ( self.StartAngleZ, self.EndAngleZ, factor )
        
        if self.timerZ >= self.DurationTimerZ then
            self.timerZ = nil
        else
            self.timerZ = self.timerZ + 1
        end
    else
        self.AngleZ = math.randomrange( self.AngleZMin - self.SaveAngleZ, self.AngleZMax - self.SaveAngleZ )
        self.SaveAngleZ = self.SaveAngleZ + self.AngleZ
        self.timerZ = 0
        
        self.StartAngleZ = self.gameObject.transform:GetEulerAngles().z
        self.EndAngleZ   = self.StartAngleZ + self.AngleZ
    end
    
    self.gameObject.physics:WarpEulerAngles( self.NewAngle )
end

function Behavior:SmoothstepX( a, b, v )
    local v = 1 - (1 - v) * (1 - v)
    local out = (b * v) + (a *(1 - v))
    return out
end

function Behavior:SmoothstepY( a, b, v )
    local v = 1 - (1 - v) * (1 - v)
    local out = (b * v) + (a *(1 - v))
    return out
end

function Behavior:SmoothstepZ( a, b, v )
    local v = 1 - (1 - v) * (1 - v)
    local out = (b * v) + (a *(1 - v))
    return out
end
```
</details>

## License
MIT
