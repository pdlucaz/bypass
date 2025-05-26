local DebugFunc = getinfo or debug.getinfo
local IsDebug = false
local hooks = {}

local DetectedMeth, KillMeth

setthreadidentity(2)

for index, value in getgc(true) do
    if typeof(value) == "table" then
        local detected = rawget(value, "Detected")
        local kill = rawget(value, "Kill")
    
        if typeof(detected) == "function" and not DetectedMeth then
            DetectedMeth = detected
            
            local hook
            hook = hookfunction(DetectedMeth, function(methodName, methodFunc, methodInfo)
                if methodName ~= "_" then
                    if IsDebug then
                        warn("Adonis Detected\nMethod: " .. tostring(methodName) .. "\nInfo: " .. tostring(methodFunc))
                    end
                end
                
                return true
            end)

            table.insert(hooks, DetectedMeth)
        end

        if rawget(value, "Variables") and rawget(value, "Process") and typeof(kill) == "function" and not KillMeth then
            KillMeth = kill
            local hook
            hook = hookfunction(KillMeth, function(killFunc)
                if IsDebug then
                    warn("Adonis tried to detect: " .. tostring(killFunc))
                end
            end)

            table.insert(hooks, KillMeth)
        end
    end
end

local hook
hook = hookfunction(getrenv().debug.info, newcclosure(function(...)
    local functionName, functionDetails = ...

    if DetectedMeth and functionName == DetectedMeth then
        if IsDebug or not IsDebug then
            print("Adonis was bypassed by the_king.78")
        end

        return coroutine.yield(coroutine.running())
    end
    
    return hook(...)
end))

setthreadidentity(7)
