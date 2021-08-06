--//Check if is opened before//--
if FLY_LOADED then
	warn("650 Fly Hack is already running!")
	return
end

pcall(function()
	getgenv().FLY_LOADED  = true 
end)

--//Load-Check//--
if not game:IsLoaded() then
	game.Loaded:Wait()
end

warn("650 Fly Hack Started!")

--//Variables//--
local Player = game:GetService("Players").LocalPlayer
local UserInputService = game:GetService("UserInputService")
local Character = Player.Character or Player.CharacterAdded:Wait()
local Humanoid = Character:WaitForChild("Humanoid")
local HumanoidRootPart = Character:WaitForChild("HumanoidRootPart")

--//Setups//--
local IsFlying,LastFly = nil,nil
local StartFlyOnExecute = true
local GoToLastLocationOnDead_WhenFlying = true
local deounce = true
local Control = {Front = 0, Back = 0, Left = 0, Right = 0}
local LastControl = {Front = 0, Back = 0, Left = 0, Right = 0}
local MaxSpeed = 50
local Speed = 0
local LastCFrame = nil

local function SendNotification(Title,Text,Duration)
	game:GetService("StarterGui"):SetCore("SendNotification",{Title = Title ; Text = Text ; Duration = Duration})
end


local function Fly(CurrentParent)
	IsFlying = true ; LastFly = true
	local BodyGyro = Instance.new("BodyGyro",CurrentParent) ; BodyGyro.P = 9e4
	BodyGyro.maxTorque = Vector3.new(math.huge, math.huge, math.huge) ; BodyGyro.cframe = CurrentParent.CFrame

	local BodyVelocity = Instance.new("BodyVelocity", CurrentParent)
	BodyVelocity.velocity = Vector3.new(0,0.1,0) ; BodyVelocity.maxForce = Vector3.new(math.huge, math.huge, math.huge)

	coroutine.resume(coroutine.create(function()
		--//Starting Loop//--
		repeat wait()
			Humanoid.PlatformStand = true

			if Control.Left + Control.Right ~= 0 or Control.Front + Control.Back ~= 0 then
				Speed = (Speed + 0.5) + (Speed/MaxSpeed)
				if Speed > MaxSpeed then
					Speed = MaxSpeed
				end
			elseif not (Control.Left + Control.Right ~= 0 or Control.Front + Control.Back ~= 0) and Speed ~= 0 then
				Speed = Speed-1
				if Speed < 0 then
					Speed = 0
				end
			end

			--Second

			if (Control.Left + Control.Right) ~= 0 or (Control.Front + Control.Back) ~= 0 then
				BodyVelocity.velocity = ((workspace.CurrentCamera.CoordinateFrame.lookVector * (Control.Front+Control.Back)) + ((workspace.CurrentCamera.CoordinateFrame * CFrame.new(Control.Left+Control.Right,(Control.Front+Control.Back)*.2,0).p) - workspace.CurrentCamera.CoordinateFrame.p))*Speed
				LastControl = {Front = Control.Front, Back = Control.Back, Left = Control.Left, Right = Control.Right}

			elseif (Control.Left + Control.Right) == 0 and (Control.Front + Control.Back) == 0 and Speed ~= 0 then
				BodyVelocity.velocity = ((workspace.CurrentCamera.CoordinateFrame.lookVector * (LastControl.Front+LastControl.Back)) + ((workspace.CurrentCamera.CoordinateFrame * CFrame.new(LastControl.Left+LastControl.Right,(LastControl.Front+LastControl.Back)*.2,0).p) - workspace.CurrentCamera.CoordinateFrame.p))*Speed
			else
				BodyVelocity.velocity = Vector3.new(0,0.1,0)
			end


			BodyGyro.CFrame = workspace.CurrentCamera.CoordinateFrame * CFrame.Angles(-math.rad((Control.Front+Control.Back)*50*Speed/MaxSpeed),0,0)

		until not IsFlying
		Control = {Front = 0, Back = 0, Left = 0, Right = 0}
		LastControl = {Front = 0, Back = 0, Left = 0, Right = 0}
		Speed = 0
		BodyGyro:Destroy() ; BodyVelocity:Destroy()
		Humanoid.PlatformStand = false
	end))

end

UserInputService.InputBegan:Connect(function(input, gameProcessed)

	if input.KeyCode == Enum.KeyCode.E then
		IsFlying = not IsFlying ; LastFly = not LastFly

		if IsFlying then
			Fly(HumanoidRootPart)
			SendNotification("650 Fly Hack","Fly Hack Is Activated",1)
		else
			SendNotification("650 Fly Hack","Fly Hack Is DeActivated",1)
		end
	end

	if input.KeyCode == Enum.KeyCode.W then
		Control.Front = 1
	elseif input.KeyCode == Enum.KeyCode.A then
		Control.Left = -1
	elseif input.KeyCode == Enum.KeyCode.S then
		Control.Back = -1
	elseif input.KeyCode == Enum.KeyCode.D then
		Control.Right = 1
	end
end)

UserInputService.InputEnded:Connect(function(input, gameProcessed)
	if input.KeyCode == Enum.KeyCode.W then
		Control.Front = 0
	elseif input.KeyCode == Enum.KeyCode.A then
		Control.Left = 0
	elseif input.KeyCode == Enum.KeyCode.S then
		Control.Back = 0
	elseif input.KeyCode == Enum.KeyCode.D then
		Control.Right = 0
	end
end)

Player.CharacterAdded:Connect(function(character)
	IsFlying = false
	local HumanoidRootPart2 = character:WaitForChild("HumanoidRootPart")
	local Humanoid2 = character:WaitForChild("Humanoid")
	Control = {Front = 0, Back = 0, Left = 0, Right = 0}
	LastControl = {Front = 0, Back = 0, Left = 0, Right = 0}
	Speed = 0
	Humanoid2.PlatformStand = false

	if LastFly then
		if GoToLastLocationOnDead_WhenFlying and LastCFrame ~= nil then
			HumanoidRootPart2.CFrame = LastCFrame
			Fly(HumanoidRootPart2)
		end
	end

	Humanoid2.Died:Connect(function()
		LastCFrame = HumanoidRootPart.CFrame
	end)

	Humanoid = Humanoid2 ; HumanoidRootPart = HumanoidRootPart2
end)

Humanoid.Died:Connect(function()
	LastCFrame = HumanoidRootPart.CFrame
end)


SendNotification("650 Fly Hack","Fly Hack Started. \n Press E To Fly!",1)

if StartFlyOnExecute then
	Fly(HumanoidRootPart)
	SendNotification("650 Fly Hack","Fly Hack Is Activated",1)
end
