## Description
* You can now have more pistols or rifles than only one
* Seperate Ammunition with each weapon

**IMPORTANT**
* This is only tested with ESX 1.9.3
* Other versions might work...

You can remove the prints :)

## Installation
### Clientside
* Go to `es_extended/server/main.lua` and search for `esx:updateWeaponAmmo` Event.
* Just add: `SetPedAmmo(GetPlayerPed(xPlayer.source), joaat(weaponName), ammoCount)`

Should look like this:
```lua
RegisterNetEvent('esx:updateWeaponAmmo')
AddEventHandler('esx:updateWeaponAmmo', function(weaponName, ammoCount)
    local xPlayer = ESX.GetPlayerFromId(source)

    if xPlayer then
        xPlayer.updateWeaponAmmo(weaponName, ammoCount)
        SetPedAmmo(GetPlayerPed(xPlayer.source), joaat(weaponName), ammoCount)
    end
end)
```
### Serverside - ESX 1.9.3
* Go to `es_extended/client/main.lua` and search for `function StartServerSyncLoops()`
* Replace this function with this one below

Should look like this:
```lua
function StartServerSyncLoops()
	if not Config.OxInventory then
		-- keep track of ammo

		local currentWeapon = {}
		for k, v in ipairs(ESX.PlayerData.loadout) do
			currentWeapon[v.name] = {}
			currentWeapon[v.name].ammo = v.ammo
			currentWeapon[v.name].name = v.name

			print('Set Weapons on StartServerSyncLoops:', currentWeapon[v.name].name, currentWeapon[v.name].ammo)
		end

		CreateThread(function()
			while ESX.PlayerLoaded do
				local sleep = 0

				if IsPedArmed(ESX.PlayerData.ped, 4) and IsPedShooting(ESX.PlayerData.ped) then
					local weaponHash = GetSelectedPedWeapon(ESX.PlayerData.ped)

					if weaponHash then
						local weapon = ESX.GetWeaponFromHash(weaponHash)

						if weapon then
							if currentWeapon[weapon.name] then
								currentWeapon[weapon.name].ammo = currentWeapon[weapon.name].ammo - 1
								TriggerServerEvent('esx:updateWeaponAmmo', weapon.name, currentWeapon[weapon.name].ammo)

								print('Update Weapon while Shooting:', currentWeapon[weapon.name].name, currentWeapon[weapon.name].ammo)
							end
						end
					end
				end

				Wait(sleep)
			end
		end)

		CreateThread(function()			
			while ESX.PlayerLoaded do
				local sleep = 1500

				-- Without this Callback the Loadout won't get updated
				ESX.TriggerServerCallback('esx:getPlayerData', function(data)
					for k, v in ipairs(data.loadout) do
						if not currentWeapon[v.name] then
							currentWeapon[v.name] = {}
							currentWeapon[v.name].ammo = v.ammo
							currentWeapon[v.name].name = v.name
				
							print('Set Weapons:', currentWeapon[v.name].name, currentWeapon[v.name].ammo)
						end
					end
				end)

				if IsPedArmed(ESX.PlayerData.ped, 4) then
					sleep = 1000
					local weaponHash = GetSelectedPedWeapon(ESX.PlayerData.ped)

					if weaponHash then
						local weapon = ESX.GetWeaponFromHash(weaponHash)

						if weapon then
							if currentWeapon[weapon.name] then
								if currentWeapon[weapon.name].ammo ~= GetAmmoInPedWeapon(ESX.PlayerData.ped, weaponHash) then
									TriggerServerEvent('esx:updateWeaponAmmo', weapon.name, currentWeapon[weapon.name].ammo)

									print('Update Weapon:', currentWeapon[weapon.name].name, currentWeapon[weapon.name].ammo)
								end
							end

						end
					end
				end

				Wait(sleep)
			end
		end)
	end
end
```
### Serverside - ESX 1.7.5 & ESX 1.6.0
* Go to `es_extended/client/main.lua` and search for `function StartServerSyncLoops()`
* Replace this function with this one below

Should look like this:
```lua
function StartServerSyncLoops()
	if not Config.OxInventory then
		-- keep track of ammo

		local currentWeapon = {}
		for k, v in ipairs(ESX.PlayerData.loadout) do
			currentWeapon[v.name] = {}
			currentWeapon[v.name].ammo = v.ammo
			currentWeapon[v.name].name = v.name

			print('Set Weapons on StartServerSyncLoops:', currentWeapon[v.name].name, currentWeapon[v.name].ammo)
		end

		CreateThread(function()
			while ESX.PlayerLoaded do
				local sleep = 0

				if IsPedArmed(ESX.PlayerData.ped, 4) and IsPedShooting(ESX.PlayerData.ped) then
					local weaponHash = GetSelectedPedWeapon(ESX.PlayerData.ped)

					if weaponHash then
						local weapon = ESX.GetWeaponFromHash(weaponHash)

						if weapon then
							if currentWeapon[weapon.name] then
								currentWeapon[weapon.name].ammo = currentWeapon[weapon.name].ammo - 1
								TriggerServerEvent('esx:updateWeaponAmmo', weapon.name, currentWeapon[weapon.name].ammo)

								print('Update Weapon while Shooting:', currentWeapon[weapon.name].name, currentWeapon[weapon.name].ammo)
							end
						end
					end
				end

				Wait(sleep)
			end
		end)

		CreateThread(function()			
			while ESX.PlayerLoaded do
				local sleep = 1500

				-- Without this Callback the Loadout won't get updated
				ESX.TriggerServerCallback('esx:getPlayerData', function(data)
					for k, v in ipairs(data.loadout) do
						if not currentWeapon[v.name] then
							currentWeapon[v.name] = {}
							currentWeapon[v.name].ammo = v.ammo
							currentWeapon[v.name].name = v.name
				
							print('Set Weapons:', currentWeapon[v.name].name, currentWeapon[v.name].ammo)
						end
					end
				end)

				if IsPedArmed(ESX.PlayerData.ped, 4) then
					sleep = 1000
					local weaponHash = GetSelectedPedWeapon(ESX.PlayerData.ped)

					if weaponHash then
						local weapon = ESX.GetWeaponFromHash(weaponHash)

						if weapon then
							if currentWeapon[weapon.name] then
								if currentWeapon[weapon.name].ammo ~= GetAmmoInPedWeapon(ESX.PlayerData.ped, weaponHash) then
									TriggerServerEvent('esx:updateWeaponAmmo', weapon.name, currentWeapon[weapon.name].ammo)

									print('Update Weapon:', currentWeapon[weapon.name].name, currentWeapon[weapon.name].ammo)
								end
							end

						end
					end
				end

				Wait(sleep)
			end
		end)
	end

	-- sync current player coords with server
	CreateThread(function()
		local previousCoords = vector3(ESX.PlayerData.coords.x, ESX.PlayerData.coords.y, ESX.PlayerData.coords.z)

		while ESX.PlayerLoaded do
			local playerPed = PlayerPedId()
			if ESX.PlayerData.ped ~= playerPed then ESX.SetPlayerData('ped', playerPed) end

			if DoesEntityExist(ESX.PlayerData.ped) then
				local playerCoords = GetEntityCoords(ESX.PlayerData.ped)
				local distance = #(playerCoords - previousCoords)

				if distance > 1 then
					previousCoords = playerCoords
					local playerHeading = ESX.Math.Round(GetEntityHeading(ESX.PlayerData.ped), 1)
					local formattedCoords = {x = ESX.Math.Round(playerCoords.x, 1), y = ESX.Math.Round(playerCoords.y, 1), z = ESX.Math.Round(playerCoords.z, 1), heading = playerHeading}
					TriggerServerEvent('esx:updateCoords', formattedCoords)
				end
			end
			Wait(1500)
		end
	end)
end
```