local RemoteEvent = script.Parent.RemoteEvent
local AimEvent = script.Parent.AimEvent
local StopEvent = script.Parent.StopEvent

local parts = {}
local affectedCharacters = {}

-- Maximum Output Spawn Function
RemoteEvent.OnServerEvent:Connect(function(player, pos)
	local char = player.Character
	if not char then return end
	local root = char:FindFirstChild("HumanoidRootPart")
	if not root then return end

	local Range = 20
	if (pos - root.Position).Magnitude > Range then
		pos = root.Position + (pos - root.Position).Unit * Range
	end

	local blue = parts[player]
	if blue then
		blue:Destroy()
	end

	local BlueAnim = player.Character.Humanoid.Animator:LoadAnimation(script.Animations.Blue)
	BlueAnim:Play()

	local CastValue = Instance.new("BoolValue")
	CastValue.Name = "CastingMaximumBlue"
	CastValue.Parent = player.Character
	task.delay(1.85, function()
		if CastValue.Parent then
			CastValue:Destroy()
		end
	end)

	local StunValue = Instance.new("BoolValue")
	StunValue.Name = "Stun"
	StunValue.Parent = player.Character

	blue = game.ReplicatedStorage.FX.MaximumOutputBlue:Clone()
	blue.Parent = workspace.FX
	blue.CFrame = CFrame.new(pos)
	blue.Anchored = true

	local bluesound = script.SoundFX.BlueSound:Clone()
	bluesound.Parent = blue
	bluesound:Play()

	for _, v in pairs(blue:GetChildren()) do
		if v:IsA("ParticleEmitter") then v.Enabled = true end
	end
	for _, v in pairs(blue.Floor:GetChildren()) do
		if v:IsA("ParticleEmitter") then v.Enabled = true end
	end
	for _, v in pairs(blue.MainAttachment:GetChildren()) do
		if v:IsA("ParticleEmitter") then v.Enabled = true end
	end

	parts[player] = blue
	affectedCharacters[player] = {}

	local pullRadius = 20
	local pullForce = 150
	local pullInterval = 0.03
	local aliveTime = 1.75
	local Damage = 0.025
	local pulling = true

    task.spawn(function()
		local elapsed = 0
		while blue and pulling and elapsed do
			elapsed += pullInterval

			if blue:FindFirstChild("CancelRelease") then
				elapsed = 0
			end

			for _, obj in pairs(workspace:GetDescendants()) do
				if obj:IsA("BasePart") and not obj.Anchored and obj ~= blue then
					local parentChar = obj:FindFirstAncestorOfClass("Model")

					if parentChar and parentChar:FindFirstChild("IFrames") then
						continue
					end

					if parentChar and player.Character and parentChar == player.Character then
						continue
					end

					local function hasDomainPrefix(char)
						if not char then return false end
						for _, child in ipairs(char:GetChildren()) do
							if child.Name:sub(1, 13) == "InsideDomain" then
								return true
							end
						end
						return false
					end

					local casterHasDomain = hasDomainPrefix(player.Character)
					local targetHasDomain = hasDomainPrefix(parentChar)

					if blue:FindFirstChild("CantPull") then
						pulling = false
						task.spawn(function()
							task.wait(1)
							for charModel, _ in pairs(affectedCharacters[player] or {}) do
								local ragdoll = charModel:FindFirstChild("RagdollTrigger")
								if ragdoll and ragdoll:IsA("BoolValue") then
									ragdoll.Value = false
								end
							end
							affectedCharacters[player] = nil
						end)
					end

					local dist = (obj.Position - blue.Position).Magnitude
					if dist <= pullRadius then
						if casterHasDomain ~= targetHasDomain then return end
						local dir = (blue.Position - obj.Position).Unit
						obj.Velocity = dir * pullForce
						if parentChar and parentChar ~= player.Character then
							local ragdoll = parentChar:FindFirstChild("RagdollTrigger")
							if ragdoll and ragdoll:IsA("BoolValue") then
								ragdoll.Value = true
								affectedCharacters[player][parentChar] = true
							end

							local humanoid = parentChar:FindFirstChildWhichIsA("Humanoid")
							if humanoid and not blue:FindFirstChild("CancelRelease") then
								humanoid:TakeDamage(Damage)
								if parentChar:FindFirstChild("Stun") == nil then
								local StunValue2 = Instance.new("BoolValue", obj:FindFirstAncestorOfClass("Model"))
								StunValue2.Name = "Stun"
								end
							end
						end
					end
				end
			end

			task.wait(pullInterval)
		end
	end)

	task.delay(aliveTime, function()
		if blue:FindFirstChild("CancelRelease") then
			return
		end

		pulling = false

		if player.Character:FindFirstChild("Stun") then
			player.Character.Stun:Destroy()
		end

		if blue and blue.Parent then
			for _, v in pairs(blue:GetChildren()) do
				if v:IsA("ParticleEmitter") then v.Enabled = false end
			end
			for _, v in pairs(blue.Floor:GetChildren()) do
				if v:IsA("ParticleEmitter") then v.Enabled = false end
			end
			for _, v in pairs(blue.MainAttachment:GetChildren()) do
				if v:IsA("ParticleEmitter") then v.Enabled = false end
			end

			local throwsound = script.SoundFX.ThrowSound:Clone()
			throwsound.Parent = blue
			throwsound:Play()
			game.Debris:AddItem(throwsound, 1)

			task.wait(0.3)
			blue:Destroy()
		end

		parts[player] = nil

		task.spawn(function()
			task.wait(1)
			for charModel, _ in pairs(affectedCharacters[player] or {}) do
				local ragdoll = charModel:FindFirstChild("RagdollTrigger")
				if ragdoll and ragdoll:IsA("BoolValue") then
					ragdoll.Value = false
				end
			end
			affectedCharacters[player] = nil
		end)
	end)
end)

-- Aim Functions
AimEvent.OnServerEvent:Connect(function(player, hit)
	local blue = parts[player]
	if blue and not blue:FindFirstChild("CancelRelease") then
		blue.CFrame = CFrame.new(hit)
	end
end)

-- Cancel Function
StopEvent.OnServerEvent:Connect(function(player)
	local blue = parts[player]
	if blue then
		local CancelRelease = Instance.new("BoolValue", blue)
		CancelRelease.Name = "CancelRelease"
		
		if player.Character:FindFirstChild("Stun") then
			player.Character:FindFirstChild("Stun"):Destroy()
		end
		
		for _, track in pairs(player.Character:FindFirstChild("Humanoid").Animator:GetPlayingAnimationTracks()) do
			if track.Animation.Name == "Blue" then
				track:Stop()
			end
		end
		
		spawn(function()
			task.wait(4)
			local CantNuke = Instance.new("BoolValue", blue)
			CantNuke.Name = "CantNuke"

			local CantPull = Instance.new("BoolValue", blue)
			CantPull.Name = "CantPull"

			for _, v in pairs(blue:GetDescendants()) do
				if v:IsA("ParticleEmitter") then
					v.Enabled = false
				end
			end
			task.wait(1)
			blue:Destroy()
		end)
	end
end)
