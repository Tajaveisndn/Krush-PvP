do getgenv().AimbotEnabled=false;getgenv().AimbotPart="Head";getgenv().AimbotSmooth=5;getgenv().FOVEnabled=false;getgenv().FOVRadius=50;getgenv().FOVColor=Color3.fromRGB(255,255,255);getgenv().ESPEnabled=false;getgenv().ESPSettings={Names=false,Boxes=false,Tracers=false,BoxColor=Color3.fromRGB(255,255,255),HeadBorderColor=Color3.fromRGB(255,0,0),HeadBorderThickness=2,TracerColor=Color3.fromRGB(255,255,255),NameColor=Color3.fromRGB(255,255,255),ShowDistance=false};getgenv().IgnoreDead=true;local Players=game:GetService("Players");local RunService=game:GetService("RunService");local UserInputService=game:GetService("UserInputService");local TweenService=game:GetService("TweenService");local Camera=workspace.CurrentCamera;local LocalPlayer=Players.LocalPlayer;local function IsValidTarget(player) if ( not player or (player==LocalPlayer)) then return false;end if ( not player.Character or  not player.Character:FindFirstChild("Humanoid")) then return false;end if (getgenv().IgnoreDead and (player.Character.Humanoid.Health<=0)) then return false;end return true;end local function IsVisible(player) local character=player.Character;if (character and character:FindFirstChild(getgenv().AimbotPart)) then local targetPart=character[getgenv().AimbotPart];local direction=(targetPart.Position-Camera.CFrame.Position).Unit;local distance=(targetPart.Position-Camera.CFrame.Position).Magnitude;local raycastParams=RaycastParams.new();raycastParams.FilterDescendantsInstances={LocalPlayer.Character};raycastParams.FilterType=Enum.RaycastFilterType.Blacklist;local raycastResult=workspace:Raycast(Camera.CFrame.Position,direction * distance ,raycastParams);if raycastResult then return raycastResult.Instance and raycastResult.Instance:IsDescendantOf(character) ;end end return false;end local function GetClosestPlayer() local closestPlayer=nil;local shortestDistance=math.huge;local centerPos=Vector2.new(Camera.ViewportSize.X/2 ,Camera.ViewportSize.Y/2 );for _,player in pairs(Players:GetPlayers()) do if IsValidTarget(player) then local character=player.Character;if (character and character:FindFirstChild(getgenv().AimbotPart)) then local targetPart=character[getgenv().AimbotPart];local pos,onScreen=Camera:WorldToViewportPoint(targetPart.Position);if onScreen then local distance=(Vector2.new(pos.X,pos.Y) -centerPos).Magnitude;if ((distance<=getgenv().FOVRadius) and (distance<shortestDistance) and IsVisible(player)) then closestPlayer=player;shortestDistance=distance;end end end end end return closestPlayer;end local ESP={Drawings={},Connections={}};ESP.CreateDrawing=function(self,type,properties) local drawing=Drawing.new(type);for property,value in pairs(properties) do drawing[property]=value;end return drawing;end;ESP.CreatePlayerESP=function(self,player) local drawings={Box=ESP:CreateDrawing("Square",{Thickness=1,Filled=false,Transparency=1,Color=getgenv().ESPSettings.BoxColor,Visible=false}),HeadMarker=ESP:CreateDrawing("Circle",{Radius=10,Filled=false,Thickness=getgenv().ESPSettings.HeadBorderThickness,Transparency=1,Color=getgenv().ESPSettings.HeadBorderColor,Visible=false}),Name=ESP:CreateDrawing("Text",{Text=player.Name,Size=13,Center=true,Outline=true,Color=getgenv().ESPSettings.NameColor,Visible=false}),Tracer=ESP:CreateDrawing("Line",{Thickness=1,Transparency=1,Color=getgenv().ESPSettings.TracerColor,Visible=false}),DistanceText=ESP:CreateDrawing("Text",{Text="",Size=13,Center=true,Outline=true,Color=Color3.fromRGB(255,255,255),Visible=false})};ESP.Drawings[player]=drawings;end;ESP.RemovePlayerESP=function(self,player) if ESP.Drawings[player] then for _,drawing in pairs(ESP.Drawings[player]) do drawing:Remove();end ESP.Drawings[player]=nil;end end;ESP.UpdateESP=function(self) for player,drawings in pairs(ESP.Drawings) do if (player and player.Character and player.Character:FindFirstChild("HumanoidRootPart") and (player~=LocalPlayer)) then local character=player.Character;local humanoidRootPart=character:FindFirstChild("HumanoidRootPart");local head=character:FindFirstChild("Head");if  not humanoidRootPart then continue;end local vector,onScreen=Camera:WorldToViewportPoint(humanoidRootPart.Position);local distance=(Camera.CFrame.Position-humanoidRootPart.Position).Magnitude;if (onScreen and getgenv().ESPEnabled) then if getgenv().ESPSettings.Boxes then local rootPos=Camera:WorldToViewportPoint(humanoidRootPart.Position);local headPos=Camera:WorldToViewportPoint(head.Position + Vector3.new(0,0.5,0) );local legPos=Camera:WorldToViewportPoint(humanoidRootPart.Position-Vector3.new(0,3,0) );local boxSize=Vector2.new((headPos.Y-legPos.Y)/2 ,headPos.Y-legPos.Y );local boxPos=Vector2.new(rootPos.X-(boxSize.X/2) ,rootPos.Y-(boxSize.Y/2) );drawings.Box.Size=boxSize;drawings.Box.Position=boxPos;drawings.Box.Visible=true;drawings.Box.Color=getgenv().ESPSettings.BoxColor;else drawings.Box.Visible=false;end local headPos=Camera:WorldToViewportPoint(head.Position);drawings.HeadMarker.Position=Vector2.new(headPos.X,headPos.Y);drawings.HeadMarker.Radius=math.clamp(500/distance ,5,10);drawings.HeadMarker.Thickness=getgenv().ESPSettings.HeadBorderThickness;drawings.HeadMarker.Visible=true;drawings.HeadMarker.Color=getgenv().ESPSettings.HeadBorderColor;if getgenv().ESPSettings.Names then drawings.Name.Position=Vector2.new(vector.X,vector.Y-40 );drawings.Name.Visible=true;drawings.Name.Color=getgenv().ESPSettings.NameColor;else drawings.Name.Visible=false;end if getgenv().ESPSettings.Tracers then drawings.Tracer.From=Vector2.new(Camera.ViewportSize.X/2 ,Camera.ViewportSize.Y);drawings.Tracer.To=Vector2.new(vector.X,vector.Y);drawings.Tracer.Visible=true;drawings.Tracer.Color=getgenv().ESPSettings.TracerColor;else drawings.Tracer.Visible=false;end if getgenv().ESPSettings.ShowDistance then drawings.DistanceText.Text=string.format("%.0f Metros",distance);drawings.DistanceText.Position=Vector2.new(vector.X,vector.Y + 35 );drawings.DistanceText.Visible=true;else drawings.DistanceText.Visible=false;end else drawings.Box.Visible=false;drawings.HeadMarker.Visible=false;drawings.Name.Visible=false;drawings.Tracer.Visible=false;drawings.DistanceText.Visible=false;end end end end;local FOVCircle=Drawing.new("Circle");FOVCircle.Visible=getgenv().FOVEnabled;FOVCircle.Radius=getgenv().FOVRadius;FOVCircle.Color=getgenv().FOVColor;FOVCircle.Thickness=1;FOVCircle.Filled=false;FOVCircle.Transparency=1;FOVCircle.Position=Vector2.new(Camera.ViewportSize.X/2 ,Camera.ViewportSize.Y/2 );local OrionLib=loadstring(game:HttpGet("https://raw.githubusercontent.com/shlexware/Orion/main/source"))();local Window=OrionLib:MakeWindow({Name="Krush PvP | Solara Hub 1.0",HidePremium=false,SaveConfig=true,ConfigFolder="SolaraConfig"});local CombatTab=Window:MakeTab({Name="Combat",Icon="rbxassetid://4483345998",PremiumOnly=false});CombatTab:AddToggle({Name="Enable Aimbot",Default=false,Callback=function(Value) getgenv().AimbotEnabled=Value;end});CombatTab:AddDropdown({Name="Aimbot Part",Default="Head",Options={"Head","HumanoidRootPart"},Callback=function(Value) getgenv().AimbotPart=Value;end});CombatTab:AddToggle({Name="FOV Circle",Default=false,Callback=function(Value) getgenv().FOVEnabled=Value;FOVCircle.Visible=Value;end});CombatTab:AddSlider({Name="FOV Size",Min=50,Max=500,Default=100,Color=Color3.fromRGB(255,255,255),Increment=1,ValueName="pixels",Callback=function(Value) getgenv().FOVRadius=Value;FOVCircle.Radius=Value;end});CombatTab:AddSlider({Name="Aimbot Smoothness",Min=1,Max=20,Default=5,Color=Color3.fromRGB(255,255,255),Increment=1,ValueName="",Callback=function(Value) getgenv().AimbotSmooth=Value;end});local ESPTab=Window:MakeTab({Name="ESP",Icon="rbxassetid://4483345998",PremiumOnly=false});ESPTab:AddToggle({Name="Enable ESP",Default=false,Callback=function(Value) getgenv().ESPEnabled=Value;end});ESPTab:AddToggle({Name="Show Names",Default=false,Callback=function(Value) getgenv().ESPSettings.Names=Value;end});ESPTab:AddToggle({Name="Show Boxes",Default=false,Callback=function(Value) getgenv().ESPSettings.Boxes=Value;end});ESPTab:AddToggle({Name="Show Tracers",Default=false,Callback=function(Value) getgenv().ESPSettings.Tracers=Value;end});ESPTab:AddToggle({Name="Show Distance",Default=false,Callback=function(Value) getgenv().ESPSettings.ShowDistance=Value;end});ESPTab:AddColorpicker({Name="Box Color",Default=Color3.fromRGB(255,255,255),Callback=function(Value) getgenv().ESPSettings.BoxColor=Value;end});ESPTab:AddColorpicker({Name="Head Border Color",Default=Color3.fromRGB(255,0,0),Callback=function(Value) getgenv().ESPSettings.HeadBorderColor=Value;end});ESPTab:AddSlider({Name="Head Border Thickness",Min=1,Max=5,Default=2,Color=Color3.fromRGB(255,255,255),Increment=1,ValueName="px",Callback=function(Value) getgenv().ESPSettings.HeadBorderThickness=Value;end});local SettingsTab=Window:MakeTab({Name="Settings",Icon="rbxassetid://4483345998",PremiumOnly=false});SettingsTab:AddButton({Name="Destroy UI",Callback=function() OrionLib:Destroy();end});OrionLib:Init();Players.PlayerAdded:Connect(function(player) ESP:CreatePlayerESP(player);end);Players.PlayerRemoving:Connect(function(player) ESP:RemovePlayerESP(player);end);for _,player in ipairs(Players:GetPlayers()) do if (player~=LocalPlayer) then ESP:CreatePlayerESP(player);end end RunService.RenderStepped:Connect(function() if getgenv().AimbotEnabled then local target=GetClosestPlayer();if (target and target.Character and target.Character:FindFirstChild(getgenv().AimbotPart)) then local targetPart=target.Character[getgenv().AimbotPart];local targetPos=targetPart.Position;local direction=(targetPos-Camera.CFrame.Position).Unit;local currentAim=Camera.CFrame.LookVector;local dot=currentAim:Dot(direction);local angle=math.acos(dot);local smoothness=getgenv().AimbotSmooth;if (angle<math.rad(5)) then smoothness=smoothness * 0.5 ;end local lerpedDirection=currentAim:Lerp(direction,1/smoothness );local targetCFrame=CFrame.new(Camera.CFrame.Position,Camera.CFrame.Position + lerpedDirection );Camera.CFrame=targetCFrame;end end if getgenv().FOVEnabled then FOVCircle.Position=Vector2.new(Camera.ViewportSize.X/2 ,Camera.ViewportSize.Y/2 );FOVCircle.Color=getgenv().FOVColor;end ESP:UpdateESP();end);local function CleanupESP() for player,_ in pairs(ESP.Drawings) do ESP:RemovePlayerESP(player);end if FOVCircle then FOVCircle:Remove();end end game:GetService("CoreGui").ChildRemoved:Connect(function(child) if (child.Name=="Orion") then CleanupESP();end end); end
