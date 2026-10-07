local player = game.Players.LocalPlayer
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local camera = game.Workspace.CurrentCamera
local playerGui = player:WaitForChild("PlayerGui")
local layoutGui = playerGui:WaitForChild('Menue', 10)
local gunGui = playerGui:WaitForChild('ScreenGui', 10)
local SoundEvent = ReplicatedStorage:WaitForChild('SoundEvent')
local LoadSlotEvent = ReplicatedStorage:WaitForChild("LoadSlot")
local currentSprintCF = CFrame.new()
local AmmoEvent = ReplicatedStorage:WaitForChild('Ammo')
local StopSound = ReplicatedStorage:WaitForChild('StopSound')
local ShootEvent = ReplicatedStorage:WaitForChild('Shoot')
local NoAmmoEvent = ReplicatedStorage:WaitForChild('NOAmmo')
local dropprd = ReplicatedStorage:WaitForChild('Dropped')
local ServerAnim = ReplicatedStorage:WaitForChild('ServerAnim')
local FireGrenade = ReplicatedStorage:WaitForChild('GrenadeEvent')


local folder = player:FindFirstChild("Folder") or Instance.new('Folder')
folder.Name = "Folder"
folder.Parent = player

local isaiming = folder:FindFirstChild("isaiming") or Instance.new('BoolValue')
isaiming.Name = 'isaiming'
isaiming.Parent = folder

local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local fireAnim = nil
local isShooting = false
local reloadThread = nil 
local playerShoot = false
local isSprint = false
local mouse = player:GetMouse()
local pickUpEvent = ReplicatedStorage:WaitForChild('PickUp')
local DestroyEvent = ReplicatedStorage:WaitForChild('Destory')



player.CharacterAdded:Connect(function(newCharacter)
	character = newCharacter
	humanoid = newCharacter:WaitForChild("Humanoid")
end)

local aimCF = CFrame.new()
local isAiming = false

local swayAMT = 0.2
local swayCF = CFrame.new()
local lastCameraCF = CFrame.new()
local mouse = player:GetMouse()

local BASE_WALKSPEED = 20

local framework = {
	inventory = {
		"TROY DEFENSE AR";
		"G19 ROLAND SPECIAL";
		"Ak-74";
		"knife";
		'Grenade';
		
	};
	module = nil;
	viewmodel = nil;
	currentSlot = 0;
	cachedAimOffset = nil;
	sound = nil;
}

local isdrop = false





local function loadAnim(viewmodelAnimator)
	if not viewmodelAnimator then
		warn("loadAnim: viewmodelAnimator is nil")
		return nil
	end

	local animId = framework.module.loadAnim
	if not animId or tostring(animId) == "" or tostring(animId) == "0" then
		warn("Warning: Load animation ID is missing or empty for " .. tostring(framework.module.GunName))
		return nil
	end

	local anim = Instance.new('Animation')
	anim.Name = 'LoadAnim'

	local animIdStr = tostring(animId)
	if not string.find(animIdStr, "rbxassetid://") then
		animIdStr = "rbxassetid://" .. animIdStr
	end

	anim.AnimationId = animIdStr
	anim.Parent = viewmodelAnimator

	local track = viewmodelAnimator:LoadAnimation(anim)
	track.Priority = Enum.AnimationPriority.Action4
	return track
end
local function NOammoAnim()
	local animController = framework.viewmodel:FindFirstChild("AnimationController") or framework.viewmodel:FindFirstChild("Humanoid")
	local viewmodelAnimator = animController and animController:WaitForChild("Animator", 1)
	if not viewmodelAnimator then
		warn("NOammoAnim: viewmodelAnimator is nil")
		return nil
	end

	-- If the empty track already exists, do not play a new one
	if framework.noAmmoTrack then
		return nil
	end

	local animId = framework.module.NOammoAnim
	if not animId or tostring(animId) == "" or tostring(animId) == "0" then
		warn("Warning: No Ammo animation ID is missing or empty for " .. tostring(framework.module.GunName))
		return nil
	end

	local anim = Instance.new('Animation')
	anim.Name = 'NoAmmo'

	local animIdStr = tostring(animId)
	if not string.find(animIdStr, "rbxassetid://") then
		animIdStr = "rbxassetid://" .. animIdStr
	end

	anim.AnimationId = animIdStr
	anim.Parent = viewmodelAnimator

	local track = viewmodelAnimator:LoadAnimation(anim)
	track.Priority = Enum.AnimationPriority.Action4
	track.Looped = false

	-- Listen for when the animation is almost finished to freeze it on the last frame
	track:GetMarkerReachedSignal("End"):Connect(function()
		track:AdjustSpeed(0) -- Freeze
	end)

	-- Fallback freeze right at the end of the animation length
	track.Stopped:Connect(function()
		if framework.module and framework.module.Ammo == 0 then
			track:Play()
			track.TimePosition = track.Length - 0.01 -- Hold the very last frame
			track:AdjustSpeed(0)
		end
	end)

	framework.noAmmoTrack = track
	return track
end



local lastServerAnimTime = 0



local function setupReloadAnimation(viewmodelAnimator)
	if not framework.module then return nil end
	local animId2 = framework.module.RealodAnim or framework.module.ReloadAnim

	if not animId2 or tostring(animId2) == "" or tostring(animId2) == "0" then
		warn("Warning: Reload animation ID is missing or empty for " .. tostring(framework.module.GunName))
		return nil
	end

	local anim2 = Instance.new('Animation')
	anim2.Name = 'Reload'

	local animIdStr = tostring(animId2)
	if not string.find(animIdStr, "rbxassetid://") then
		animIdStr = "rbxassetid://" .. animIdStr
	end

	anim2.AnimationId = animIdStr
	anim2.Parent = framework.viewmodel

	local track = viewmodelAnimator:LoadAnimation(anim2)
	track.Priority = Enum.AnimationPriority.Action4
	return track
end

local function loadSlot(slotNum)
	if reloadThread then
		task.cancel(reloadThread)
		StopSound:FireServer()
		reloadThread = nil
	end

	local slotName = framework.inventory[slotNum]
	if not slotName then return end

	local moduleFolder = ReplicatedStorage:WaitForChild('Modules')
	local viewModelFolder = ReplicatedStorage:WaitForChild('Viewmodels')

	local targetviewmodel = viewModelFolder:FindFirstChild(slotName)
	local targetmodule = moduleFolder:FindFirstChild(slotName)

	if not targetviewmodel or not targetmodule then 
		warn("Missing assets for slot: " .. slotName)
		return 
	end

	if framework.reloadTrack then framework.reloadTrack:Stop() end
	if framework.fireTrack then framework.fireTrack:Stop() end

	if framework.viewmodel ~= nil then
		framework.viewmodel:Destroy()
		framework.viewmodel = nil
	end

	for i, v in pairs(camera:GetChildren()) do
		if v:IsA('Model') then
			v:Destroy()
		end
	end

	framework.viewmodel = targetviewmodel:Clone()
	framework.viewmodel.Parent = camera
	framework.module = require(targetmodule)
	framework.sound = framework.module.sound
	LoadSlotEvent:FireServer(slotName)

	local animObject = Instance.new('Animation')
	animObject.Name = 'Fire'

	local animId = tostring(framework.module.fireAnim)
	if not string.find(animId, "rbxassetid://") then
		animId = "rbxassetid://" .. animId
	end
	animObject.AnimationId = animId
	animObject.Parent = framework.viewmodel

	local animController = framework.viewmodel:FindFirstChild("AnimationController") or framework.viewmodel:FindFirstChild("Humanoid")
	local viewmodelAnimator = animController and animController:WaitForChild("Animator", 1)

	if viewmodelAnimator then
		framework.fireTrack = viewmodelAnimator:LoadAnimation(animObject)
		if framework.fireTrack then
			framework.fireTrack.Priority = Enum.AnimationPriority.Action4
			framework.fireTrack.Looped = false
		end

		framework.reloadTrack = setupReloadAnimation(viewmodelAnimator)
	else
		warn("Could not find Animator inside viewmodel for " .. slotName)
	end
	
	
	
	framework.loadTrack = loadAnim(viewmodelAnimator)
	if framework.loadTrack then
		framework.loadTrack:Play()
	end

	-- Cache the aim offset once so animations can't affect it
	if framework.viewmodel:FindFirstChild('AimPart') and framework.viewmodel.PrimaryPart then
		framework.cachedAimOffset = framework.viewmodel.AimPart.CFrame:ToObjectSpace(framework.viewmodel.PrimaryPart.CFrame)
	end
	framework.currentSlot = slotNum
	isAiming = false
	isaiming.Value = false
	aimCF = CFrame.new()

	-- ريست لجميع حالات الحركة عند تغيير السلاح لمنع التعليق
	playerShoot = false
	isShooting = false
	isSprint = false
	isdrop = false
	if humanoid then humanoid.WalkSpeed = BASE_WALKSPEED end
	game.ReplicatedStorage.EquipEvent:FireServer(slotName)
	
	
	
	
	if framework.reloadTrack then framework.reloadTrack:Stop() end
	if framework.fireTrack then framework.fireTrack:Stop() end
	if framework.noAmmoTrack then 
		framework.noAmmoTrack:Stop() 
		framework.noAmmoTrack = nil 
	end
end



local canRealod = true

local function fireWeapon()
	-- 1. Grenade Handling (Slot 5)
	if framework.currentSlot == 5 then
		local grenadeModule = require(ReplicatedStorage.Modules.Grenade)

		if grenadeModule and grenadeModule.Ammo > 0 and canRealod then
			grenadeModule.Ammo -= 1
			canRealod = false -- Prevent spamming click during throw

			if framework.fireTrack then
				framework.fireTrack:Play()

				local connection
				connection = framework.fireTrack:GetMarkerReachedSignal("GranadeThrow"):Connect(function()
					connection:Disconnect()

					print("Grenade fired to server!")
					FireGrenade:FireServer(mouse.Hit.Position)

					if framework.viewmodel then
						local grenadeBody = framework.viewmodel:FindFirstChild('1')
						if grenadeBody then grenadeBody:Destroy() end

						local pin = framework.viewmodel:FindFirstChild('Pin')
						if pin then pin:Destroy() end
					end
				end)
			end
		end
		return
	end

	-- 2. Regular Guns Firing Logic
	if framework.fireTrack and framework.sound and framework.module and framework.module.Ammo > 0 then
		if framework.currentSlot ~= 4 then
			framework.module.Ammo -= 1
		end

		framework.fireTrack:Play()
		SoundEvent:FireServer(framework.currentSlot)

		local damage = framework.module.damage
		local headshot = framework.module.headshot
		local muzzlePart = framework.viewmodel:FindFirstChild('Muzzle')

		if muzzlePart then
			ShootEvent:FireServer(muzzlePart.CFrame, mouse.Hit.Position, damage, headshot)
		end

		if tick() - lastServerAnimTime >= 0.15 then
			lastServerAnimTime = tick()
			ServerAnim:FireServer(framework.currentSlot, framework.module.GunName)
		end
	end
end

RunService.RenderStepped:Connect(function()
	local rot = camera.CFrame:ToObjectSpace(lastCameraCF)
	local X, Y, Z = rot:ToOrientation()
	local currentSwayAmount = isAiming and (swayAMT * 0.2) or swayAMT

	swayCF = swayCF:Lerp(CFrame.Angles(math.sin(X) * currentSwayAmount, math.sin(Y) * currentSwayAmount, 0), 0.1)

	local ammoText = gunGui:FindFirstChild('Ammo')
	local totalAmmoText = gunGui:FindFirstChild('TotalAmmo')
	local gunNameText = gunGui:FindFirstChild('GunName')
	local fireModeText = gunGui:FindFirstChild('FireMode')

	if framework.module and not isdrop and framework.currentSlot ~= 4  then
		if ammoText then ammoText.Text = framework.module.Ammo end
		if totalAmmoText then totalAmmoText.Text = framework.module.MaxAmmo end
		if gunNameText then gunNameText.Text = framework.module.GunName or "" end
		if fireModeText and framework.module.currentFireMode then 
			fireModeText.Text = framework.module.currentFireMode 
		end
	elseif isdrop then
		if ammoText then ammoText.Text = "0" end
		if totalAmmoText then totalAmmoText.Text = "0" end
		if gunNameText then gunNameText.Text = "Drop" end
		if fireModeText then fireModeText.Text = "" end
	elseif framework.currentSlot == 4 then
		if ammoText then ammoText.Text = "" end
		if fireModeText then fireModeText.Text = "" end
		if ammoText then ammoText.Text = "" end
		if totalAmmoText then totalAmmoText.Text = "" end
		if gunNameText then gunNameText.Text = "Knife" end
	end

	lastCameraCF = camera.CFrame

	if character and humanoid and framework.viewmodel and framework.module and framework.viewmodel.PrimaryPart then
		local targetSprintCF = CFrame.new()

		if humanoid.WalkSpeed >= 30 and not isAiming and framework.module.sprintCF and not playerShoot then
			targetSprintCF = framework.module.sprintCF
		end

		local lerpSpeed = framework.module.aimSmooth or 0.1
		currentSprintCF = currentSprintCF:Lerp(targetSprintCF, lerpSpeed)

		if isAiming and framework.module.canAim and framework.cachedAimOffset then
			local offset = framework.cachedAimOffset
			aimCF = aimCF:Lerp(offset, framework.module.aimSmooth)
		else
			local offset = CFrame.new()
			aimCF = aimCF:Lerp(offset, framework.module.aimSmooth)
		end

		local bobDivider = isAiming and 12 or 6
		local boboffset = CFrame.new(
			-humanoid.CameraOffset.X / bobDivider, 
			-humanoid.CameraOffset.Y / bobDivider, 
			-humanoid.CameraOffset.Z / bobDivider
		)

		-- Prevent fire animation from recoiling the gun body (keeps sight stable)
		if framework.fireTrack and framework.fireTrack.IsPlaying and framework.viewmodel and framework.viewmodel.PrimaryPart then
			local mainMotor = framework.viewmodel.PrimaryPart:FindFirstChild("Main")
			if mainMotor and mainMotor:IsA("Motor6D") then
				mainMotor.Transform = CFrame.new()
			end
		end

		framework.viewmodel:PivotTo(camera.CFrame * swayCF * aimCF * currentSprintCF * boboffset)
	end
	
	-- تحديث سرعة اللاعب بناءً على الحالات بشكل مباشر ومنظم لتفادي التعليق التلقائي
	if humanoid then
		if isAiming or playerShoot then
			humanoid.WalkSpeed = BASE_WALKSPEED
		elseif isSprint then
			humanoid.WalkSpeed = 30
		else
			humanoid.WalkSpeed = BASE_WALKSPEED
		end
	end
	
	
	
	
	

	
	
	
end)



local UserInputService = game:GetService("UserInputService")
local dropWeaponRemote = ReplicatedStorage:WaitForChild('dropWeaponRemote')

local menue, Background, loadout1, loadout2, primary, pistol, select
local num = 0
local weapons = {}

local function fps()
	player.CameraMode = Enum.CameraMode.LockFirstPerson

end

local function setup()
	-- YOUR EXACT LINES (using menue directly instead of layoutGui):
	menue = playerGui:WaitForChild('Menue', 10)
	Background = menue:WaitForChild('Frame'):WaitForChild('Background')
	loadout1 = Background:WaitForChild('Loadout1')
	loadout2 = Background:WaitForChild('Loadout2')
	primary = Background:WaitForChild('PrimaryGun')
	pistol = Background:WaitForChild('Pistol')
	select = Background:WaitForChild('Select')
	task.defer(function()
		player.CameraMode = Enum.CameraMode.Classic
		player.CameraMinZoomDistance = 12
		player.CameraMaxZoomDistance = 12
		UserInputService.MouseIconEnabled = true
		UserInputService.MouseBehavior = Enum.MouseBehavior.Default
	end)
	-- UI Reset & Setup
	menue.Enabled = true
	primary.Text = 'TROY DEFENSE AR'
	num = 1

	player.CameraMode = Enum.CameraMode.Classic
	UserInputService.MouseIconEnabled = true

	-- Button Listeners
	loadout1.MouseButton1Click:Connect(function()
		primary.Text = 'TROY DEFENSE AR'
		num = 1
	end)

	loadout2.MouseButton1Click:Connect(function()
		primary.Text = 'AK-74'
		num = 2
	end)

	select.MouseButton1Click:Connect(function()
		if num == 1 then
			loadSlot(1)
			table.insert(weapons, 1)
			task.wait(1)
			loadSlot(2)
			table.insert(weapons, 2)

			menue.Enabled = false
			fps()

		elseif num == 2 then
			loadSlot(3)
			table.insert(weapons, 3)
			task.wait(1)
			loadSlot(2)
			table.insert(weapons, 2)

			menue.Enabled = false
			fps()
		end
	end)
end

-- Triggers on spawn and respawn
player.CharacterAdded:Connect(function(char)
	setup()
end)

if player.Character then
	setup()
end


UserInputService.InputBegan:Connect(function(input, gameProcessed)
	if gameProcessed or not humanoid or humanoid.Health <= 0 then return end

	if input.KeyCode == Enum.KeyCode.One then
		local primarySlot = (num == 1) and 1 or 3
		if framework.inventory[primarySlot] then
			loadSlot(primarySlot)
		end
	end

	if input.KeyCode == Enum.KeyCode.Two then
		if framework.inventory[2] then
			loadSlot(2)
		end
	elseif input.KeyCode == Enum.KeyCode.Three then
		loadSlot(4)
	elseif input.KeyCode == Enum.KeyCode.Y then
		local grenadeModule = require(ReplicatedStorage.Modules.Grenade)

		if grenadeModule then
			-- Case A: You already have a grenade loaded in Ammo, just equip it!
			if grenadeModule.Ammo > 0 then
				canRealod = true
				loadSlot(5)

				-- Case B: Active Ammo is empty, but you have reserve MaxAmmo -> reload and equip
			elseif grenadeModule.Ammo == 0 and grenadeModule.MaxAmmo > 0 then
				task.defer(function()
					task.wait(0.2)
					if grenadeModule.MaxAmmo > 0 and grenadeModule.Ammo == 0 then
						grenadeModule.MaxAmmo -= 1
						grenadeModule.Ammo = 1
						canRealod = true
						loadSlot(5)
					end
				end)
			else
				warn("No grenades remaining in Ammo or MaxAmmo!")
			end
		end
	

	elseif input.KeyCode == Enum.KeyCode.LeftShift and not isAiming then
		isSprint = true	
		if humanoid then humanoid.WalkSpeed = 30 end -- Set speed directly here

	elseif input.UserInputType == Enum.UserInputType.MouseButton2 and not isdrop then
		UserInputService.MouseIconEnabled = false
		isAiming = true
		isaiming.Value = true 
		if humanoid then humanoid.WalkSpeed = BASE_WALKSPEED end -- Force slow down when aiming

	elseif input.KeyCode == Enum.KeyCode.V and not isdrop then
		if framework.module and framework.module.availableModes then
			local modes = framework.module.availableModes
			local current = framework.module.currentFireMode

			if current == 'Semi' and table.find(modes,'Auto') then
				framework.module.currentFireMode = 'Auto'
			elseif current == 'Auto' and table.find(modes,'Semi') then
				framework.module.currentFireMode = 'Semi'
			end
		end

	elseif input.KeyCode == Enum.KeyCode.R and not isdrop then
		if not framework.module or isShooting or reloadThread ~= nil then return end
		if framework.noAmmoTrack then
			framework.noAmmoTrack:Stop()
			framework.noAmmoTrack:Destroy()
		end
  
		local currentModule = framework.module
		if framework.currentSlot == 1  and not isdrop then
			local ammoNeeded = 30 - currentModule.Ammo
			if ammoNeeded > 0 and currentModule.MaxAmmo > 0 then
				AmmoEvent:FireServer("Realod")
				if framework.reloadTrack then framework.reloadTrack:Play() end

				reloadThread = task.spawn(function()
					task.wait(2)
					if framework.module == currentModule and humanoid.Health > 0 then 
						local bulletsToLoad = math.min(ammoNeeded, currentModule.MaxAmmo)
						currentModule.MaxAmmo -= bulletsToLoad
						currentModule.Ammo += bulletsToLoad
					end
					reloadThread = nil
				end)
			end

		elseif framework.currentSlot == 2 and not isdrop then
			local ammoNeeded = 17 - currentModule.Ammo
			if ammoNeeded > 0 and currentModule.MaxAmmo > 0 then
				AmmoEvent:FireServer("Realod")
				if framework.reloadTrack then framework.reloadTrack:Play() end

				reloadThread = task.spawn(function()
					task.wait(2)
					if framework.module == currentModule and humanoid.Health > 0 then
						local bulletsToLoad = math.min(ammoNeeded, currentModule.MaxAmmo)
						currentModule.MaxAmmo -= bulletsToLoad
						currentModule.Ammo += bulletsToLoad
						
						
						if framework.noAmmoTrack then
							framework.noAmmoTrack:Stop()
							framework.noAmmoTrack = nil
						end
					end
					reloadThread = nil
				end)
			end

		elseif framework.currentSlot == 3 and not isdrop then
			local ammoNeeded = 60 - currentModule.Ammo
			if ammoNeeded > 0 and currentModule.MaxAmmo > 0 then
				AmmoEvent:FireServer("Realod")
				if framework.reloadTrack then framework.reloadTrack:Play() end

				reloadThread = task.spawn(function()
					task.wait(2)
					if framework.module == currentModule and humanoid.Health > 0 then
						local bulletsToLoad = math.min(ammoNeeded, currentModule.MaxAmmo)
						currentModule.MaxAmmo -= bulletsToLoad
						currentModule.Ammo += bulletsToLoad
					end
					reloadThread = nil
				end)
			end
		end
		
	
		-- 1. Ensure a weapon is currently equipped and frame exists
	elseif input.KeyCode == Enum.KeyCode.P then
		-- 1. Ensure a weapon is currently equipped and valid
		if framework and framework.viewmodel and framework.module then
			local gunName = framework.module.GunName
			local droppedSlot = framework.currentSlot

			DestroyEvent:FireServer()

			-- 2. Destroy the local viewmodel and clear weapon tracks
			framework.viewmodel:Destroy()
			framework.viewmodel = nil
			framework.module = nil
			framework.fireTrack = nil
			framework.reloadTrack = nil
			framework.loadTrack = nil
			framework.noAmmoTrack = nil
			framework.cachedAimOffset = nil

			-- 3. REMOVE the dropped weapon from inventory so key 1/2 cannot reload it
			if droppedSlot > 0 then
				framework.inventory[droppedSlot] = nil
			end
			framework.currentSlot = 0
			isdrop = false

			-- 4. Notify Server to drop physical model in Workspace
			dropWeaponRemote:FireServer(gunName)
		end


	elseif input.UserInputType == Enum.UserInputType.MouseButton1 and not isdrop then
		if not framework.module or reloadThread ~= nil then return end
		

		if framework.module.Ammo == 0 and not isdrop then
			-- Tell the server we have no ammo (for server-side click sounds, etc.)

			-- Play the animation locally immediately
			if framework.currentSlot == 2 then
				local noAmmoTrack = NOammoAnim()
				if noAmmoTrack then
					noAmmoTrack:Play()

				end
			end
			
			return 
		end
		if framework.currentSlot == 4 then
			local Event = ReplicatedStorage:WaitForChild('PlayerShoot')
		
			Event:FireServer()
		end
		
		if framework.currentSlot == 5 then
			
			
			
			
			
		end
		playerShoot = true

		local mode = framework.module.currentFireMode or "Semi"
		local rate = framework.module.fireRate or 0.1

		if mode == 'Semi' then
			fireWeapon()

			playerShoot = false 
		elseif mode == 'Auto' then
			isShooting = true

			task.spawn(function()
				while isShooting and framework.module and framework.module.currentFireMode == 'Auto' and humanoid.Health > 0 do
					if framework.module.Ammo <= 0 or reloadThread ~= nil then 
						break 
					end
					fireWeapon()

					task.wait(rate)
				end
				isShooting = false
				playerShoot = false
			end)
		end
	end
end)

UserInputService.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton2 then
		UserInputService.MouseIconEnabled = true
		isAiming = false
		isaiming.Value = false
		-- Return to sprint speed if still holding shift, otherwise normal speed
		if humanoid then
			humanoid.WalkSpeed = isSprint and 30 or BASE_WALKSPEED
		end

	elseif input.KeyCode == Enum.KeyCode.LeftShift then
		isSprint = false
		if humanoid then humanoid.WalkSpeed = BASE_WALKSPEED end

	elseif input.UserInputType == Enum.UserInputType.MouseButton1 then
		isShooting = false
		playerShoot = false
	end
end)
mouse.Icon = 'rbxassetid://16549986336'

pickUpEvent.OnClientEvent:Connect(function(gunName)
	local targetSlot = nil
	if gunName == 'Ak-74' then
		targetSlot = 3
	elseif gunName == 'TROY DEFENSE AR' then
		targetSlot = 1
	elseif gunName == 'G19 ROLAND SPECIAL' then
		targetSlot = 2
	end
	
	print(targetSlot)
	framework.inventory[targetSlot] = gunName
	loadSlot(targetSlot)
end)
