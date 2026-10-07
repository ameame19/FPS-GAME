local replicatedStorage = game:GetService('ReplicatedStorage')
local soundService = game:GetService('SoundService')

-- Remotes
local SoundEvent = replicatedStorage:WaitForChild('SoundEvent')
local AmmoEvent = replicatedStorage:WaitForChild('Ammo')
local StopSound = replicatedStorage:WaitForChild('StopSound')
local ShootEvent = replicatedStorage:WaitForChild('Shoot')
local NoAmmoEvent = replicatedStorage:WaitForChild('NOAmmo')
local pickUpEvent = replicatedStorage:WaitForChild('PickUp')
local DestroyEvent = replicatedStorage:WaitForChild('Destory')
local ServerAnim = replicatedStorage:WaitForChild('ServerAnim')
local EquipEvent = replicatedStorage:WaitForChild('EquipEvent') 
local dropWeaponRemote = replicatedStorage:WaitForChild("dropWeaponRemote")
local grenadeDrop = replicatedStorage:WaitForChild('Dropped'):WaitForChild('Grenade')

-- Assets
local sound2 = soundService:WaitForChild('Gunshot')
local sound = soundService:WaitForChild('smgfire')
local aksound = soundService:WaitForChild('ak74shot')
local weaponsFolder = replicatedStorage:WaitForChild('Weapons')
local droppedFolder = replicatedStorage:WaitForChild("Dropped")
local knifeSwing = soundService:WaitForChild('Knife swing')
local FireGrenade = replicatedStorage:WaitForChild('GrenadeEvent')
local Explosion = replicatedStorage:WaitForChild('Dropped'):WaitForChild('Explosion')
local SoundExploasion = soundService:WaitForChild('grenade explosion')
local newsound = SoundExploasion:Clone()


local activeIdleTracks = {}
local activeFireTracks = {}
local activeWeaponModels = {}

EquipEvent.OnServerEvent:Connect(function(player, item)
	local character = player.Character
	if not character then return end

	if activeWeaponModels[player] then
		activeWeaponModels[player]:Destroy()
		activeWeaponModels[player] = nil
	end

	if activeIdleTracks[player] then
		activeIdleTracks[player]:Stop()
		activeIdleTracks[player] = nil
	end

	local weaponTemplate = weaponsFolder:FindFirstChild(item)
	if weaponTemplate then
		local gun = weaponTemplate:Clone()
		gun.Parent = character
		activeWeaponModels[player] = gun

		local rightHand = character:FindFirstChild("RightHand") or character:FindFirstChild("Right Arm")
		if rightHand and gun.PrimaryPart then
			local joint = Instance.new("Motor6D")
			joint.Name = "WeaponWeld"
			joint.Part0 = rightHand
			joint.Part1 = gun.PrimaryPart

			if item == 'TROY DEFENSE AR' then
				joint.C0 = CFrame.new(0, -0.03, 0.1) * CFrame.Angles(math.rad(-90), math.rad(180), 0)
			elseif item == 'G19 ROLAND SPECIAL' then
				joint.C0 = CFrame.new(0, -0.15, 0.1) * CFrame.Angles(math.rad(-90), math.rad(-90), 0)
			elseif item == 'Ak-74' then
				joint.C0 = CFrame.new(0, -0.4, 0.1) * CFrame.Angles(math.rad(-90), math.rad(-90), 0)
			end

			joint.Parent = gun
		end

		local moduleScript = replicatedStorage:WaitForChild('Modules'):FindFirstChild(item)
		if moduleScript then
			local weaponData = require(moduleScript)
			local AnimId = weaponData.IdleAnim

			if AnimId then
				local hum = character:FindFirstChildOfClass('Humanoid')
				if hum then
					local anim = Instance.new('Animation')
					local idString = string.gsub(tostring(AnimId), "%s+", "")

					if string.match(idString, "^%d+$") then
						anim.AnimationId = "rbxassetid://" .. idString
					else
						anim.AnimationId = idString
					end

					local animator = hum:FindFirstChildOfClass("Animator") or Instance.new("Animator", hum)
					local track = animator:LoadAnimation(anim)
					track.Priority = Enum.AnimationPriority.Action
					track:Play()
					activeIdleTracks[player] = track
				end
			end
		end
	end
end)
local Debris = game:GetService("Debris")

local MAX_DISTANCE = 70
local THROW_POWER = 40
local EXPLOSION_DAMAGE = 80

FireGrenade.OnServerEvent:Connect(function(plr, mousePos)
	local char = plr.Character
	if not char then return end

	local humR = char:FindFirstChild("HumanoidRootPart")
	if not humR then return end

	-- Spawn location
	local spawnPos = humR.Position + (humR.CFrame.LookVector * 2) + Vector3.new(0, 1, 0)

	-- Calculate directional vector clamped to MAX_DISTANCE
	local rawDirection = (mousePos - spawnPos)
	local distance = math.min(rawDirection.Magnitude, MAX_DISTANCE)

	local flatDirection = Vector3.new(rawDirection.X, 0, rawDirection.Z)
	local lookVector = flatDirection.Magnitude > 0 and flatDirection.Unit or humR.CFrame.LookVector

	-- Clone grenade
	local grenade = grenadeDrop:Clone()

	if grenade:IsA("Model") then
		grenade:PivotTo(CFrame.lookAt(spawnPos, spawnPos + lookVector))
	else
		grenade.CFrame = CFrame.lookAt(spawnPos, spawnPos + lookVector)
	end

	grenade.Parent = workspace

	-- Get physics part
	local mainPart = grenade:IsA("Model") and (grenade.PrimaryPart or grenade:FindFirstChildWhichIsA("BasePart")) or grenade

	if mainPart then
		mainPart.CanCollide = true
		mainPart.Anchored = false

		-- Upward velocity increases when aiming higher
		local upwardBias = math.clamp(rawDirection.Y * 0.5 + 20, 10, 45)

		-- Throw force calculation
		local forwardForce = lookVector * (THROW_POWER + (distance * 0.4))
		local upwardForce = Vector3.new(0, upwardBias, 0)

		mainPart.AssemblyLinearVelocity = forwardForce + upwardForce
	end

	-- Fuse Timer
	task.delay(4, function()
		if grenade and grenade.Parent and mainPart then
			-- 1. Freeze motion instantly so position cannot shift
			mainPart.Anchored = true
			mainPart.AssemblyLinearVelocity = Vector3.zero

			-- 2. Get exact current position of the grenade part
			local explodePos = mainPart.Position
			
			

			-- 3. Create explosion EXACTLY at the grenade position
			local exp = Instance.new("Explosion")
			exp.Position = explodePos
			exp.BlastRadius = 15
			exp.BlastPressure = 500000
			newsound.Parent = exp
			newsound:Play()

			-- Damage nearby humanoids
			local hitHumanoids = {}
			exp.Hit:Connect(function(part)
				local targetChar = part.Parent
				local targetHum = targetChar and targetChar:FindFirstChildOfClass("Humanoid")

				if targetHum and not hitHumanoids[targetHum] then
					hitHumanoids[targetHum] = true
					targetHum:TakeDamage(EXPLOSION_DAMAGE)
				end
			end)

			exp.Parent = workspace

			-- 4. Destroy the grenade model immediately when exploding
			grenade:Destroy()
		end
	end)
end)

StopSound.OnServerEvent:Connect(function(plr)
	local char = plr.Character
	if not char or not char:FindFirstChild("HumanoidRootPart") then return end

	-- Find and stop any active sounds attached to the player's HumanoidRootPart
	for _, child in ipairs(char.HumanoidRootPart:GetChildren()) do
		if child:IsA("Sound") then
			child:Stop()
			child:Destroy()
		end
	end
end)

--------------------------------------------------------------------------------
-- 2. DESTROY / DROP CLEANUP EVENT
--------------------------------------------------------------------------------
DestroyEvent.OnServerEvent:Connect(function(player)
	if activeWeaponModels[player] then
		activeWeaponModels[player]:Destroy()
		activeWeaponModels[player] = nil
	end

	if activeIdleTracks[player] then
		activeIdleTracks[player]:Stop()
		activeIdleTracks[player] = nil
	end
end)

--------------------------------------------------------------------------------
-- 3. SERVER SHOOT ANIMATION
--------------------------------------------------------------------------------
ServerAnim.OnServerEvent:Connect(function(plr, currentSlot, GunName)
	local char = plr.Character
	if not char then return end

	local hum = char:FindFirstChildOfClass('Humanoid')
	if not hum or hum.Health <= 0 then return end

	local moduleScript = replicatedStorage:WaitForChild('Modules'):FindFirstChild(GunName)
	if not moduleScript then return end

	local weaponData = require(moduleScript)
	local fireAnimId = weaponData.fireServer or weaponData.fireAnim

	if not fireAnimId or tostring(fireAnimId) == "" or tostring(fireAnimId) == "0" then return end

	local animIdStr = tostring(fireAnimId)
	if not string.find(animIdStr, "rbxassetid://") then
		animIdStr = "rbxassetid://" .. animIdStr
	end

	local animator = hum:FindFirstChildOfClass('Animator') or Instance.new('Animator', hum)

	if not activeFireTracks[plr] or activeFireTracks[plr].Name ~= GunName then
		local anim = Instance.new('Animation')
		anim.AnimationId = animIdStr

		local track = animator:LoadAnimation(anim)
		track.Priority = Enum.AnimationPriority.Action2
		track.Looped = false
		activeFireTracks[plr] = { Track = track, Name = GunName }
	end

	local activeTrack = activeFireTracks[plr].Track
	if activeTrack then
		activeTrack:Stop()
		activeTrack:Play(0.05, 1, 1)
	end
end)

--------------------------------------------------------------------------------
-- 4. PHYSICAL DROP SYSTEM
--------------------------------------------------------------------------------
dropWeaponRemote.OnServerEvent:Connect(function(player, gunName)
	local character = player.Character
	if not character or not character:FindFirstChild("HumanoidRootPart") then return end

	local weaponTemplate = droppedFolder:FindFirstChild(gunName)
	if weaponTemplate then
		local droppedModel = weaponTemplate:Clone()
		droppedModel.Parent = workspace

		local spawnCFrame = character.HumanoidRootPart.CFrame * CFrame.new(0, 0, -5)
		droppedModel:PivotTo(spawnCFrame)

		for _, part in ipairs(droppedModel:GetDescendants()) do
			if part:IsA("BasePart") then
				part.CanCollide = true
				part.Anchored = false
			end
		end

		local proxy = Instance.new("ProximityPrompt")
		proxy.ObjectText = gunName
		proxy.ActionText = "Pick Up"
		proxy.HoldDuration = 0.2
		proxy.MaxActivationDistance = 8
		proxy.Parent = droppedModel.PrimaryPart or droppedModel:FindFirstChildWhichIsA("BasePart")

		proxy.Triggered:Connect(function(plr)
			pickUpEvent:FireClient(plr, gunName)
			droppedModel:Destroy()
		end)
	end
end)

--------------------------------------------------------------------------------
-- 5. AUDIO & SHOOTING EVENTS
--------------------------------------------------------------------------------
SoundEvent.OnServerEvent:Connect(function(plr, SlotNum)
	local char = plr.Character
	if not char or not char:FindFirstChild("HumanoidRootPart") then return end

	local targetSound = (SlotNum == 1 and sound) or (SlotNum == 2 and sound2) or (SlotNum == 3 and aksound)
	if targetSound then
		local newSound = targetSound:Clone()
		newSound.Parent = char.HumanoidRootPart
		newSound:Play()
		newSound.Ended:Connect(function()
			newSound:Destroy()
		end)
	end
end)

AmmoEvent.OnServerEvent:Connect(function(plr, Name, currentSlot)
	local char = plr.Character
	if not char or not char:FindFirstChild("HumanoidRootPart") then return end

	local noAmmo = soundService:FindFirstChild(Name)
	if noAmmo then
		local new = noAmmo:Clone()
		new.Parent = char.HumanoidRootPart
		new:Play()
		game.Debris:AddItem(new, 2)
	end
end)
local event = replicatedStorage:WaitForChild('PlayerShoot')
event.OnServerEvent:Connect(function(plr)
	local char = plr.Character
	if not char or not char:FindFirstChild("HumanoidRootPart") then return end

	local root = char.HumanoidRootPart

	local part = Instance.new('Part')
	part.Size = Vector3.new(1, 1, 2) 

	-- Position relative to the player's current character orientation:
	part.CFrame = root.CFrame * CFrame.new(1, 0.5, -1.5)
	part.Anchored = true
	part.CanCollide = false
	part.CanTouch = true -- Explicitly enable touch detection
	part.Transparency = 1
	knifeSwing:Play()
	
	part.Parent = workspace

	
	local hasDealtDamage = false

	part.Touched:Connect(function(hit)
		if not hit  or hit:IsDescendantOf(char) then return end

		local targetChar = hit:FindFirstAncestorOfClass("Model")
		if targetChar then
			local humanoid = targetChar:FindFirstChildOfClass("Humanoid")
			if humanoid then
				local targetPlayer = game.Players:GetPlayerFromCharacter(targetChar)
				if targetPlayer and targetPlayer == plr then return end

				
				
					humanoid:TakeDamage(10)
				
			end
		end

	end)
		

	game.Debris:AddItem(part,1)

end)
ShootEvent.OnServerEvent:Connect(function(plr, muzzleCFrame, mousePos, damage, headshot,currentSlot)
	local char = plr.Character
	if not char then return end
	
	

	local bullet = Instance.new('Part')
	bullet.Name = 'Bullet'
	bullet.Size = Vector3.new(0.1, 0.1, 2)
	bullet.Material = Enum.Material.Neon
	bullet.Color = Color3.fromRGB(243, 255, 23)
	bullet.CanCollide = false
	bullet.Anchored = false
	bullet.CFrame = CFrame.new(muzzleCFrame.Position, mousePos)
	bullet.Parent = workspace

	game.Debris:AddItem(bullet, 1)

	local BodyVelocity = Instance.new('BodyVelocity')
	BodyVelocity.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
	BodyVelocity.Velocity = bullet.CFrame.LookVector * 1000 -- Increased speed for realistic bullet travel
	BodyVelocity.Parent = bullet

	bullet:SetNetworkOwner(plr)

	bullet.Touched:Connect(function(hit)
		if not hit or hit:IsDescendantOf(bullet) or hit:IsDescendantOf(char) then return end

		local targetChar = hit:FindFirstAncestorOfClass("Model")
		if targetChar then
			local humanoid = targetChar:FindFirstChildOfClass("Humanoid")
			if humanoid then
				local targetPlayer = game.Players:GetPlayerFromCharacter(targetChar)
				if targetPlayer and targetPlayer == plr then return end

				if hit.Name == 'Head' then
					humanoid:TakeDamage(headshot)
				else
					humanoid:TakeDamage(damage)
				end
			end
		end

		bullet:Destroy()
	end)
end)

-- Player Removal Cleanup
game.Players.PlayerRemoving:Connect(function(plr)
	activeIdleTracks[plr] = nil
	activeFireTracks[plr] = nil
	activeWeaponModels[plr] = nil
end)
