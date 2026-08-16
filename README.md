# UltraGraphics
UltraGraphics/Config.lua













-- 
-- Copyright 2026 Fr4nsson. All rights reserved.
-- Redistribution or derivative works require written permission.
-- Personal use and private modification of this file are permitted.
-- 
-- ============================================================
-- UltraGraphics - UE4SS Lua mod for Palworld by Fr4nsson
--  * Applies verified Engine.ini-style CVar overrides from config.lua
--  * Overrides World Partition LoadingRange via console command
--  * Overrides post process component settings
--  * Overrides light components (on sync + as they spawn)
--  * Creates additional point-light instances when configured
--  * Mirrors color/visibility between configured light instances
--  * Configures static-mesh shadow casting on matched light actors
--  * Disables Palworld's point-light culling for specific lights
-- Mostly diff-based: ordinary properties are written only when different.
-- Packed light-shadow flags are reapplied through native setters because their
-- reflected values are unreliable in some UE4SS builds.
--
-- WorldPartition: runs the engine console command
-- wp.Runtime.OverrideRuntimeSpatialHashLoadingRange
-- (game default range 25600) so distant terrain/LODs
-- stream in further away.
--
-- Config is automatically applied on world load and teleport.
-- You can also edit this config and press the default F8 key
-- to immediately apply the updated settings in-game.
--
-- Most Enabled toggles only control whether UltraGraphics applies a category
-- or entry; already-written property values are not restored when disabled.
-- CVars and point-light culling are exceptions: pressing F8 restores saved
-- pre-mod CVar values for removed/disabled assignments and re-registers lights
-- that are no longer selected so vanilla point-light culling resumes.
-- ============================================================

local Config = {
    -- Log levels per category:
    --   0 = warnings only
    --   1 = info  (short "Patched X" message when something changed)
    --   2 = debug (logs every individual value that changed)
    LogLevel = {
        CVars = 0,
        WorldPartition = 0,
        PostProcess = 0,
        DisableLightCulling = 0,
        Lights = 0, -- fires per spawned light, keep 0 unless developing
    },
    
    -- UE4SS key reference:
    -- https://docs.ue4ss.com/lua-api/table-definitions/key.html#key-code-strings
    -- (changing keybind settings requires a game restart)
    Keys = {
        Sync = {
            Enabled = true,
            Key = "F8", -- re-sync changes made in this config
        },
        ToggleLumenDiffuse = {
            Enabled = true,
            Key = "F9", -- toggles r.Lumen.DiffuseIndirect.Allow
        },
        TogglePerformanceMode = {
            Enabled = true,
            Key = "F10", -- toggles the Performance section below
        },
    },

    -- ============================================================
    -- PERFORMANCE MODE
    -- ============================================================
    --
    -- The authored default for performance mode. Pressing the F10 hotkey
    -- overrides it and remembers your choice in UltraGraphics_State.ini, next
    -- to this file, so the mode survives restarts. This mod never writes to
    -- config.lua: the value below stays exactly as you set it, and it applies
    -- again as soon as the saved override matches it (or you delete the state
    -- file). Keep it on its own line.
    PerformanceMode = false,

    -- Everything performance mode changes lives here and is fully editable.
    -- Performance mode always resolves LAST so its values stick:
    --   * CVars below are merged over the main CVars block, overriding any
    --     assignment it shares a name with.
    --   * Light distances are written after templates, actor Settings and
    --     instance Settings have all been merged.
    -- Turning performance mode off restores the values these replaced.
    Performance = {
        -- Applied to every configured point light EXCEPT the FastTravel and
        -- Collectibles groups, so towers, map points, notes and effigies stay
        -- visible from a distance.
        --
        -- A light that already configures a SHORTER non-zero distance keeps its
        -- own value, because it is already cheaper than the limit. A longer
        -- distance, an unset distance, or 0 (which means "never cull") is
        -- replaced by the limit. Switching performance mode off puts the
        -- configured value back, or 0 where the light configures none.
        Lights = {
            -- Use one of these presets, 1, 2, 3 or 4 or make a custom one
            
            -- Draw Distance 1
            --MaxDrawDistance = 2000,
            --MaxDistanceFadeRange = 666,
            
            -- Draw Distance 2
            MaxDrawDistance = 3000,
            MaxDistanceFadeRange = 1000,
            
            -- Draw Distance 3
            --MaxDrawDistance = 4000,
            --MaxDistanceFadeRange = 1333,
            
            -- Draw Distance 4
            --MaxDrawDistance = 5000,
            --MaxDistanceFadeRange = 1666,
        },

        -- The sun's ray-traced distance field shadow distance, lowered back to
        -- the vanilla value. Only applied when the configured distance in the
        -- BP_PalSkyCreator_C entry is higher than this.
        SunDistanceFieldShadowDistance = 25000.0, -- Vanilla: 25000.0

        -- Same format as the main CVars block above; ';' and '#' comments and
        -- inline comments all work the same way.
        --
        -- NOTE ON sg.* SCALABILITY GROUPS: setting one of these applies a whole
        -- bucket of individual r.* values at once. The mod therefore always
        -- executes every sg.* assignment BEFORE the individual r.* overrides,
        -- both when enabling and when restoring, so the individual values below
        -- are never silently overwritten. Their position in this list does not
        -- matter.
        CVars = [=[
# CVars in this section are applied when performance mode is ENABLED.
# ULTRAGRAPHICS_PERFORMANCE_CVAR_OVERRIDES
# Scalability groups. Applied before the individual overrides below
# 0 = Low
# 1 = Medium
# 2 = High
# 3 = Epic
# 4 = Cinematic
# Uncomment below to use
# sg.EffectsQuality=0
# sg.FoliageQuality=0
# sg.GlobalIlluminationQuality=0
# sg.LandscapeQuality=0
# sg.PostProcessQuality=0
# sg.ReflectionQuality=0
# sg.ShadingQuality=0
# sg.ShadowQuality=1
# sg.TextureQuality=2
# sg.ViewDistanceQuality=2

# Resolution Scale
# Any value set here overrides the resolution scale applied by the in-game DLSS setting.
# The values below are provided for reference.
# DLSS Off               = 100
# DLSS Quality           = 66.7
# DLSS Balanced          = 58
# DLSS Performance       = 50   # Lowest available in-game settings.
# DLSS Ultra Performance = 33.3
r.ScreenPercentage=33.3

# Screen space global illumination
r.SSGI.Enable=0
r.SSGI.Quality=0

# Ambient occlusion
# r.DistanceFieldAO=0
r.AmbientOcclusionLevels=0
r.AOQuality=1

# Screen space reflections
r.SSR.Quality=1
r.SSR.HalfResSceneColor=1

# Static-mesh and landscape LOD pop-in while moving around
r.StaticMeshLODDistanceScale=1.0
r.LandscapeLODDistributionScale=1.5
r.LandscapeLOD0DistributionScale=1.75

# Fog
# r.Fog=0

# Volumetric fog resolution
# r.VolumetricFog=0
r.VolumetricFog.GridPixelSize=16
r.VolumetricFog.GridSizeZ=64

# Shadow and draw distance
r.ViewDistanceScale=1
r.Shadow.CSM.MaxCascades=2
r.Shadow.MaxCSMResolution=1024
r.Shadow.MaxResolution=1024
grass.DisableDynamicShadows=1

# Lumen tracing budget
r.Lumen.TraceDistanceScale=0.5
r.Lumen.ScreenProbeGather.MaxRayIntensity=4
r.Lumen.ScreenProbeGather.Temporal.MaxFramesAccumulated=32
r.Lumen.ScreenProbeGather.TracingOctahedronResolution=16
r.Lumen.ScreenProbeGather.RadianceCache.ProbeResolution=16
r.Lumen.ScreenProbeGather.DownsampleFactor=32
r.Lumen.ScreenProbeGather.TraceMeshSDFs=0
r.Lumen.TraceMeshSDFs=0
r.Lumen.TraceMeshSDFs.Allow=0
r.Lumen.TranslucencyVolume.Enable=0
r.LumenScene.FarField=0

# Lumen Reflections
r.ReflectionMethod=2
r.Lumen.Reflections.DownsampleFactor=2
r.Lumen.TranslucencyReflections.FrontLayer.Enable=0
r.Lumen.TranslucencyReflections.FrontLayer.Allow=0
r.TranslucencyLightingVolumeDim=8
        ]=],
    },
    
    -- Paste a complete Engine.ini (or just name=value lines) between the
    -- [=[ and ]=] markers. Only [ConsoleVariables] and [SystemSettings]
    -- assignments are executed. Other sections, blank lines, and comments are
    -- ignored. Both ';' and '#' comments are supported, including inline:
    --     r.Shadow.MaxCSMResolution=3072 ; Double vanilla on Epic
    --
    -- The first readable runtime value before UltraGraphics changes each CVar
    -- is retained for this game session. Removing an assignment or setting
    -- Enabled=false and pressing F8 restores that saved value and verifies
    -- the read-back. The current outstanding values are also written to
    -- UltraGraphics_CVarOriginalValues.txt next to this config for inspection.
    CVars = {
        Enabled = true,
        VerifyDelayMs = 100,
        EngineIni = [=[
# CVars in this section are applied when performance mode is DISABLED.
# ULTRAGRAPHICS_CVAR_OVERRIDES
# https://dev.epicgames.com/documentation/unreal-engine/unreal-engine-console-variables-reference?lang=en-US

# Ultimate Engine Tweaks enables instance occlusion culling.
# To maintain compatibility with users who use it, we disable it here because leaving it on will
# cause dynamic foliage shadows to disappear or flicker at certain viewing distances and/or angles.
r.InstanceCulling.OcclusionCull=0

# Extends the object draw distance.
r.ViewDistanceScale=4

# Remove camera/post-processing blur
r.FilmGrain=0
r.SceneColorFringe.Max=0
r.SceneColorFringeQuality=0

# Texture clarity
r.MaxAnisotropy=16

# Overall sharpening
r.Tonemapper.Sharpen=1.25

# Reduces static-mesh and landscape LOD pop-in while moving around
r.StaticMeshLODDistanceScale=0.1
r.LandscapeLODDistributionScale=4
r.LandscapeLOD0DistributionScale=4

# Allows Medium shadow quality settings in-game while preserving distance field shadows.
# Useful for local point lights with bUseRayTracedDistanceFieldShadows=True enabled,
# If torch self-shadowing is distracting, enable bUseRayTracedDistanceFieldShadows
# for that light in config.lua to remove the self-shadow effect.
# Performance impact is minimal, so this is recommended to keep enabled.
r.DistanceFieldShadowing=1

# Keeps volumetric fog enabled when lowering shadow quality.
# Normally, setting shadows to Medium in-game disables volumetric fog, 
# thus removing fog lighting, light shafts, and atmospheric effects.
# Disable with r.VolumetricFog=0 for better performance in fog-heavy scenes,
# at the cost of reduced atmosphere and lighting quality.
# Recommended to keep enabled as it significantly improves overall image quality.
r.VolumetricFog=1

# Enables Lumen global illumination.
r.DynamicGlobalIlluminationMethod=1

# Lumen settings.
# Enables Lumen diffuse indirect lighting.
# This is also used as the main Lumen toggle in the script.
r.Lumen.DiffuseIndirect.Allow=1

# Reduces volumetric fog by like 75% if enabled so I keep it off for now, 
# need to experiment more with it.
r.Lumen.TranslucencyVolume.Enable=0

# Enables screen-space tracing for detailed visible geometry and foliage.
# Improves fine indirect-shadow and ambient-occlusion detail, particularly on
# thin or complex geometry that may not be represented accurately by Lumen's
# lower-detail scene-tracing fallbacks.
r.Lumen.ScreenProbeGather.ScreenTraces=1

# Disable hierarchical depth-buffer traversal if it produces excessive noise
r.Lumen.ScreenProbeGather.ScreenTraces.HZBTraversal=1
r.Lumen.ScreenProbeGather.ScreenTraces.HZBTraversal.SkipFoliageHits=1
r.Lumen.ScreenProbeGather.ScreenTraces.HZBTraversal.SkipHairHits=0
r.Lumen.ScreenProbeGather.ScreenTraces.HZBTraversal.MaxIterations=32
r.Lumen.ScreenProbeGather.ScreenTraces.HZBTraversal.RelativeDepthThickness=0.005

# Allows mesh-distance-field tracing
r.Lumen.TraceMeshSDFs=1
r.Lumen.TraceMeshSDFs.Allow=1

# Disables mesh-SDF fallback specifically for Screen Probe Gather.
# In Palworld, enabling it brightens trees and produces worse foliage shading.
r.Lumen.ScreenProbeGather.TraceMeshSDFs=0

# Global SDF reconstruction
# Controls how much the Global SDF expands thin surfaces in regular covered regions. 
# Lower value reduces over-occlusion and consequently brighten covered interiors, 
# but can also cause more light leaking.
r.LumenScene.GlobalSDF.CoveredExpandSurfaceScale=1   # Default: 1
# Applies to regions containing only two-sided mesh distance fields, which frequently means foliage.
# Player-built interiors are also darkened because their modular pieces appear
# to rely more heavily on this not-covered Global SDF path. Static interiors
# placed in the world are generally represented by covered distance fields and
# are therefore affected mainly by CoveredExpandSurfaceScale instead. 
r.LumenScene.GlobalSDF.NotCoveredExpandSurfaceScale=0.6   # Default: 0.6

# Lumen Screen Probe Gather quality and stability.
# Limits excessively bright indirect-light samples and firefly artifacts.
r.Lumen.ScreenProbeGather.MaxRayIntensity=4
# Accumulates more temporal history for smoother and more stable indirect 
# lighting, at the cost of slower lighting response and possible ghosting.
r.Lumen.ScreenProbeGather.Temporal.MaxFramesAccumulated=32
# Resolution of the tracing octahedron. Determines how many traces are done per probe.
r.Lumen.ScreenProbeGather.TracingOctahedronResolution=16
# Pixel size of the screen tile that a screen probe will be placed on.
r.Lumen.ScreenProbeGather.DownsampleFactor=16

# Explicitly disables Lumen Hardware Ray Tracing and forces the 
# less expensive software tracing path. Added for clarity and to 
# ensure the intended setting is applied. The shipped Palworld build 
# does not include Hardware Ray Tracing support.
r.Lumen.HardwareRayTracing=0

# =================================================================
# REFLECTIONS
# =================================================================
# Reflection settings, too high MaxRoughnessToTrace will cause metals to go black
r.ReflectionMethod=1
r.Lumen.Reflections.DownsampleFactor=1
r.Lumen.Reflections.ScreenTraces=1
r.Lumen.Reflections.MaxRayIntensity=2.5
r.Lumen.Reflections.Temporal.MaxFramesAccumulated=8
r.Lumen.Reflections.SmoothBias=0.35
r.Lumen.Reflections.MaxRoughnessToTrace=0.25
r.Lumen.Reflections.RoughnessFadeLength=0.25
r.Lumen.Reflections.HierarchicalScreenTraces.MaxIterations=128
r.Lumen.Reflections.HierarchicalScreenTraces.RelativeDepthThickness=0.1
# Uncomment to get glass reflections
# the reflections is not that great, that is why it is off by default
# r.Lumen.TranslucencyReflections.FrontLayer.Enable=1
# r.Lumen.TranslucencyReflections.FrontLayer.Allow=1

# Stops glass from glowing
# Dimensions of the volume textures used for translucency lighting. 
# Larger textures result in higher resolution but lower performance.
r.TranslucencyLightingVolumeDim=8   # Default 64
# Distance from the camera that the volume cascade should end
r.TranslucencyLightingVolumeInnerDistance=1   # Default 1500
r.TranslucencyLightingVolumeOuterDistance=2   # Default 5000

# =================================================================
# AMBIENT_OCCLUSION
# =================================================================
# We disable r.Lumen.ScreenProbeGather.ScreenSpaceBentNormal
# because the AO is noisy, we instead use r.Lumen.DiffuseIndirect.SSAO
# alternatively one could use
# r.Lumen.ScreenProbeGather.ScreenSpaceBentNormal.ApplyDuringIntegration=1
# but then reflective metals will turn dark
# 
# Whether to compute a short range, full resolution AO.
r.Lumen.ScreenProbeGather.ScreenSpaceBentNormal=0
# Whether to render and apply SSAO to Lumen GI.
r.Lumen.DiffuseIndirect.SSAO=1
# Allows to scale the ambient occlusion radius (SSAO).
# r.AmbientOcclusionRadiusScale=1   # Default: 1

# =================================================================
# SHADOWS
# =================================================================
r.Shadow.CSM.MaxCascades=4

# Cache CSM updates to reduce CPU/GPU cost
r.Shadow.CSMCaching=1

# Disable DLSS motion vector dilation
r.NGX.DLSS.DilateMotionVectors=0

# Directional Shadow Resolution (Directional Light - Sun, Moon)
# r.Shadow.MaxCSMResolution=512
# r.Shadow.MaxCSMResolution=1024
# r.Shadow.MaxCSMResolution=1536   # Palworld default, no matter the in-game shadow setting
# r.Shadow.MaxCSMResolution=2048
# r.Shadow.MaxCSMResolution=3072   # Double Palworld default
# r.Shadow.MaxCSMResolution=4096
# r.Shadow.MaxCSMResolution=8192   # Very expensive

# Local Shadow Resolution (Point & Spot Lights, i.e.
# Torches, Campfires, Fireplaces, Lanterns, Lamps, etc.)
# r.Shadow.MaxResolution=512
# r.Shadow.MaxResolution=1024
# r.Shadow.MaxResolution=1536   # Palworld default, no matter the in-game shadow setting
# r.Shadow.MaxResolution=2048
# r.Shadow.MaxResolution=3072   # Double Palworld default
# r.Shadow.MaxResolution=4096
# r.Shadow.MaxResolution=8192

# =================================================================
# HIGH_QUALITY_SHADOWS
# =================================================================
# Enables Virtual Shadow Maps for significantly improved shadow quality.
# You need to set r.Shadow.Virtual.Enable=1 to enable.
#
# Benefits:
# - More detailed shadows
# - Softer shadow edges
# - Shadows render at much greater distances
#
# These settings are subjective and may require adjustment depending
# on your hardware and personal preference. If you experience shadow
# flickering or instability, try adjusting the values or reverting to
# the default settings. This is why these options are not enabled by
# default.
#
# Enables Virtual Shadow Maps, providing significantly higher-quality shadows
# than Palworld's default setup and allowing them to render much farther away.
# Here is some sites if you want to read more about Virtual Shadow Maps
# https://www.strayspark.studio/blog/virtual-shadow-map-optimization-open-worlds-ue5-7
# https://dev.epicgames.com/documentation/unreal-engine/virtual-shadow-maps-in-unreal-engine
r.Shadow.Virtual.Enable=0    # SET THIS TO 1 TO ENABLE HIGH QUALITY SHADOWS. DO NOT POST ISSUES REGARDING SHADOWS IF YOU HAVE ENABLED THIS.

# Number of SMRT rays traced per shadow sample.
# Higher values reduce noise and improve soft shadow quality at a GPU cost.
# r.Shadow.Virtual.SMRT.RayCountLocal=8
# r.Shadow.Virtual.SMRT.RayCountDirectional=8

# Number of shadow map samples taken along each SMRT ray.
# Higher values improve soft shadow accuracy and stability at a GPU cost.
# r.Shadow.Virtual.SMRT.SamplesPerRayLocal=2
# r.Shadow.Virtual.SMRT.SamplesPerRayDirectional=4

# Virtual Shadow Resolution
# -2 = 4x resolution (~16x page demand)
# -1 = 2x resolution (~4x page demand)
#  0 = 1x resolution
#  1 = 1/2 resolution (~1/4 page demand)
#  2 = 1/4 resolution (~1/16 page demand)
# r.Shadow.Virtual.ResolutionLodBiasLocal=2                # Epic:  0.0
# r.Shadow.Virtual.ResolutionLodBiasDirectional=0          # Epic: -1.5
# r.Shadow.Virtual.ResolutionLodBiasLocalMoving=2          # Epic:  1.0 # Does not work in palworld
# r.Shadow.Virtual.ResolutionLodBiasDirectionalMoving=-1   # Epic: -1.5 # Does not work in palworld

# Physical page pool. ~64 KB per page => 8192 ≈ 512 MB (more with
# separate static caching). Overflow causes checkerboard corruption
# and missing shadows
# r.Shadow.Virtual.MaxPhysicalPages=8192   # Epic: 4096, Cinematic: 8192

# Prevents distant shadows from disappearing too early.
# This issue can be seen, for example, inside the first cave where
# the player spawns.
r.Shadow.Virtual.UseFarShadowCulling=0

# Preserves shadows on small objects such as bushes.
r.Shadow.RadiusThreshold=0.01

# Removes thin black lines caused by low-positioned lights hitting
# the floor at a shallow angle, exposing Virtual Shadow Map sampling
# and self-shadowing precision errors.
r.Shadow.Virtual.ResolutionLodBiasLocal=0
r.Shadow.Virtual.NormalBias=1
        ]=],
    },

    WorldPartition = {
        -- Vanilla values:
        -- WorldPartitionRuntimeSpatialHash /Game/Pal/Maps
        -- 0 MainGrid     25 600
        -- 1 Foliage      25 600
        -- 2 FarMountain 102 400
        -- 3 Oilrig      102 400
        -- 4 CloseRange   18 000
        
        -- Increase the detailed MainGrid streaming range to reduce aggressive
        -- World Partition HLOD swaps.
        -- This is the most performance-intensive setting in the file because:
        -- The game must load, stream, and render more of the world.
        -- The mod must scan every loaded actor when it synchronizes.
        -- Loading screens and region streaming may take longer.
        -- More lights remain active at the same time.
        -- If loading becomes slow or your frame rate drops, restore LoadingRange to 25600 before changing anything else.
        -- The cost increases quickly beyond 51200. A value of 76800 loads roughly nine times the land area of the default value, not three times.
        -- [WORLD_PARTITION_MASTER_SWITCH] <-- search it to jump here
        Enabled = false, -- WARNING: Read the above before you activate

        -- Delay after world load before applying LoadingRange.
        WORLD_PARTITION_DELAY_MS = 1000,
        
        -- Palworld's detailed MainGrid is grid 0.
        GridIndex = 0,
        
        -- Vanilla is 25600 (256 metres). 51200 doubles the 
        -- radius and cover roughly four times the ground area.
        LoadingRange = 51200,
    },
    
    -- Overrides FPostProcessSettings on live components matched by
    -- class + name substrings (ALL NameContains entries must match).
    -- ============================================================
    -- Post Process
    -- ============================================================
    PostProcess = {
        Enabled = true,
        SkyCreator = {
            Enabled = true,
            ComponentClass = "PostProcessComponent",
            NameContains = {
                "PPSkyCreator_",
                ".Post Process Component"
            },
            Settings = {
                --bOverride_AutoExposureMinBrightness = true,        -- DO NOT SET HERE IF YOU ARE USING ULTRA WEATHER MOD, SET IT THERE INSTEAD
                --bOverride_AutoExposureMaxBrightness = true,        -- DO NOT SET HERE IF YOU ARE USING ULTRA WEATHER MOD, SET IT THERE INSTEAD
                --AutoExposureMinBrightness = 0.30, -- Vanilla: 0.50 -- DO NOT SET HERE IF YOU ARE USING ULTRA WEATHER MOD, SET IT THERE INSTEAD
                --AutoExposureMaxBrightness = 3.40, -- Vanilla: 3.40 -- DO NOT SET HERE IF YOU ARE USING ULTRA WEATHER MOD, SET IT THERE INSTEAD
                
                --bOverride_LocalExposureHighlightContrastScale = true,
                --LocalExposureHighlightContrastScale = 1.00, -- Vanilla: 1.00 -- DO NOT SET HERE IF YOU ARE USING ULTRA WEATHER MOD, SET IT THERE INSTEAD
                
                --bOverride_LocalExposureShadowContrastScale = false,
                --LocalExposureShadowContrastScale = 1.00, -- Vanilla: 1.00
                
                --bOverride_LocalExposureDetailStrength = false,
                --LocalExposureDetailStrength = 1.00, -- Vanilla: 1.00
                
                --bOverride_LocalExposureMiddleGreyBias = false,
                --LocalExposureMiddleGreyBias = -1.00, -- Vanilla: 0.00

                bOverride_LumenSkylightLeaking = true,
                LumenSkylightLeaking = 0.10, -- Vanilla: 0.00, less dark interiors with lumen, allow some ambient light to bleed through
                
                bOverride_LumenDiffuseColorBoost = true,
                LumenDiffuseColorBoost = 1.50, -- Vanilla: 1.00
                
                bOverride_LumenFullSkylightLeakingDistance = true,
                LumenFullSkylightLeakingDistance = 10000.0,
            }
        },
    },
    
    -- Keyed by ACTOR CLASS NAME. Applied to existing actors on sync
    -- and to new ones the moment they spawn/stream in (BeginPlay hook).
    -- NameContains filters light components WITHIN the matched actor. Every
    -- listed substring must be present in the component's full name (AND, not OR).
    -- Group is config-only metadata. The normalization step at the end of this
    -- file combines each entry's Enabled value with Lights.Groups[Group].
    Lights = {
        -- ============================================================
        -- Light controls
        -- ============================================================
        
        -- Master switch for every configured light.
        -- [LIGHT_MASTER_SWITCH] <-- search it to jump here
        Enabled = true,
        
        -- Global switch for disabling Palworld's point-light culling.
        -- Only light entries with DisableLightCulling = true are affected.
        --
        -- By default, only light entries whose actor class starts with
        -- "BP_BuildObject" use DisableLightCulling = true.
        --
        -- This section and every per-light DisableLightCulling value are
        -- reloaded when you press F8. Newly selected visible lights are
        -- unregistered from Palworld's culling manager; lights no longer selected
        -- are re-registered so vanilla culling resumes without a restart.
        DisableLightCulling = {
            -- [CULLING_MASTER_SWITCH] <-- search it to jump here
            Enabled = true,

            -- Optional global draw-distance overrides. F8 applies changed
            -- non-nil values to affected lights. nil preserves the current value;
            -- it does not restore a value previously written by this option.
            -- Large values can make far-away lights expensive.
            --MaxDrawDistance = nil,      -- example: 100000
            --MaxDistanceFadeRange = nil, -- example: 10000
        },
        
        -- Category switches. These are reloaded when F8 is pressed.
        -- Disabling a group keeps its entries in the returned flat Lights table,
        -- but forces them off. This lets main.lua clean up mod-created lights and
        -- restore vanilla point-light culling without requiring a restart.
        -- [LIGHT_GROUPS] <-- search it to jump here
        Groups = {
            Environment = true,   -- Sun and global/environment lights
            Player = true,        -- Torch and player equipment
            Weapons = true,       -- Muzzle flashes, missiles and other weapon related lights
            PalSpheres = true,    -- Palspheres in Hand, Thrown, and as Loot in World
            Pals = true,          -- Foxparks, Rooby, Helzephyr etc...
            BuildObjects = true,  -- Player-buildable lights, lamps, fires and torches etc
            SkillEffects = true,  -- Pal skills, like Ignis Blast, BP_SkillEffect_*
            FastTravel = true,    -- Towers and map points
            WorldEvents = true,   -- Supply drop, meteor etc...
            Dungeons = true,      -- Entrances, exits and dungeon-specific objects
            Collectibles = true,  -- Notes, Lifmunk effigies etc...
            Settlements = true,   -- Settlement torches, braziers and candelabras
            WorldObjects = true,  -- Jump pads, chests and miscellaneous objects
            NPCObjects = true,    -- NPC and faction campfires, lamps and torches
        },
        
        -- ============================================================
        -- LIGHT TEMPLATES AND INSTANCE OVERRIDES
        -- ============================================================
        --
        -- Light templates reduce repeated configuration while keeping each actor's
        -- setup explicit. Reusable templates are defined under:
        --
        --     Lights.Templates
        --
        -- An actor applies templates by listing their names in order:
        --
        --     Templates = {
        --         "PointLightDefaults",
        --         "CreatedPointLight",
        --         "LightColorCyan",
        --     },
        --
        -- Templates are merged from TOP TO BOTTOM. If multiple templates define the
        -- same value, the template listed later wins. The actor entry is merged last,
        -- so fields and Settings written directly on the actor override all templates.
        --
        -- Merge priority, from lowest to highest:
        --
        --     first template
        --         -> later templates
        --             -> actor fields and actor Settings
        --                 -> instance Settings
        --
        -- Templates do not inherit or automatically apply other templates. Every
        -- template used by an actor must be listed explicitly on that actor. This keeps
        -- inherited behavior visible and avoids hidden inheritance chains.
        --
        -- Normal nested tables are deep-merged. This allows an actor to override one
        -- field inside Settings while retaining other fields supplied by templates.
        --
        -- Ordered arrays such as Templates, NameContains and Instances replace earlier
        -- arrays completely rather than merging item by item.
        --
        -- ACTOR FAMILIES
        --
        -- ActorClasses lets one config entry expand into several exact actor-class
        -- entries before templates and group switches are applied. Every listed class
        -- receives the family's fields, Templates and Settings. Optional Overrides can
        -- supply final per-class differences. The family name itself is removed before
        -- config.lua returns, so main.lua still receives a flat actor-class table.
        --
        -- Keep actor metadata such as Enabled and Group on the actor entry instead of
        -- inside templates. Templates may provide shared fields such as:
        --
        --     ComponentClass
        --     CreateIfMissing
        --     CreateEagerly
        --     DisableLightCulling
        --     IgnoreLightCurve
        --     Settings
        --
        -- CreateEagerlyExplanation
        -- 
        -- Some lights like the ones created for Skill effects or pal spheres 
        -- spawn, fire and despawn within a second or two.
        -- Without this the light is created about half a second after
        -- the effect appears, so the effect can be half over, or already
        -- gone, before it lights up.
        -- CreateEagerly creates the light during the actor's own spawn
        -- instead. If that turns out to be too early the entry quietly
        -- falls back to the normal timing, so this can only make a light
        -- appear sooner, never later.
        --
        -- Works on any entry, whether it creates its own light or only patches
        -- one Palworld already owns. For a created light it means the light
        -- exists during the spawn; for a patched one it means the settings are
        -- written then, rather than about half a second later.
        --
        -- If the light is not there yet at that point, an entry that creates
        -- its own falls back to the normal timing, and one that only patches
        -- waits for the light to turn up on its own. Either way this can only
        -- make a light appear sooner, never later.
        --
        -- Do not add CreateIfMissing to an entry that only patches an existing
        -- light just to get this. It is not needed, and it would let the mod
        -- create a second light of its own if the real one happens not to be
        -- attached yet when the actor spawns.
        --
        -- Worth adding to any other short-lived actor, such as thrown
        -- spheres or projectiles. Long-lived actors like towers and lamps
        -- do not need it, since half a second while a region streams in
        -- is not that noticeable.
        --
        -- CreateEagerlyBeforeAndAfter
        --
        -- Before, the light was queued with a settle of two pump ticks at
        -- 250 ms each, and the real delay depended on where the spawn landed
        -- between ticks:
        --
        --   Spawn just after a tick fires, next tick at +250 ms, the one
        --   after at +500 ms, so 500 ms.
        --   Spawn just before a tick fires, that tick fires immediately, the
        --   next at +250 ms, so 250 ms.
        --
        -- So 250 to 500 ms, averaging about 375 ms, worst case 500 ms.
        --
        -- After, the light is created during the spawn itself, so it appears in
        -- the same frame rather than up to 500 ms later. Measured at 0.72 ms of
        -- work for 25 thrown Ultimate Pal spheres, so the delay is effectively
        -- gone: 500 ms down to under a millisecond.
        --
        -- A thrown sphere only lives about 0.8 to 1.6 seconds, so that delay
        -- used to eat a large part of the visible flight:
        --
        -- +----------------------------------------+---------------------+------------+
        -- | Scenario                               | Light appears after | Flight lit |
        -- +----------------------------------------+---------------------+------------+
        -- | Before, worst (500ms delay, 0.8s life) | 500ms               | ~37%       |
        -- | Before, typical (375ms, 1.3s)          | 375ms               | ~71%       |
        -- | After                                  | 0.7ms               | >99.9%     |
        -- +----------------------------------------+---------------------+------------+
        --
        -- The true worst case was worse than 500 ms. If that first attempt
        -- was not ready it re-queued for another three ticks, landing at
        -- 1250 ms, by which point a short lived actor has already despawned,
        -- so the light never appeared at all and nothing was logged about it.
        --
        -- CreateEagerlyCost
        --
        -- The light is built on the game thread while the actor is spawning,
        -- so the work lands in the frame that spawns it rather than being
        -- spread out by the normal deferred pass.
        --
        -- Measured at about 0.7 ms for an Ultimate Pal sphere. That entry is
        -- unusual in creating two lights, writing around thirty properties
        -- between them, so roughly 0.35 ms per created light. Every other
        -- sphere and most skill effects create a single light and cost about
        -- half as much, near 0.35 ms.
        --
        -- The per light figure is the one to estimate with: roughly 0.35 ms per
        -- created light, so a one light entry around 0.35 ms, a two light entry
        -- around 0.7 ms, a four light entry around 1.4 ms. Repeat runs of 25
        -- throws varied between 0.56 and 0.76 ms for the same two light sphere,
        -- so treat these as ballpark figures rather than exact costs.
        --
        -- That is nothing for one actor at a time, which is the normal case
        -- for thrown spheres and most skill effects. It matters when many
        -- copies of the same actor spawn in the same frame. Ten single light
        -- actors at once is around 3.5 ms, ten two light actors around 7 ms,
        -- against the 16 ms available for a 60 fps frame. So for anything that
        -- spawns in volleys, such as multi projectile or scattershot skills,
        -- prefer leaving CreateEagerly off and accepting the small delay, or
        -- keep the entry's Instances and Settings small, since the cost scales
        -- with the number of lights and properties.
        --
        -- These figures are absolute CPU work and are not tied to frame rate.
        -- The creation runs start to finish inside the actor's spawn without
        -- waiting on a frame boundary, so capping frames does not make it
        -- cheaper. Runs of 25 throws at 30, 60 and 120 fps gave 0.56, 0.72 and
        -- 0.76 ms for the same two light sphere. That spread is within the
        -- measurement noise of 25 samples and is not a frame rate effect.
        --
        -- What does change with frame rate is how much of the budget it takes.
        -- A frame is 33 ms at 30 fps, 16 ms at 60 and 8 ms at 120, so the same
        -- 0.7 ms is about 2 percent of a frame at 30 fps and about 9 percent at
        -- 120. Ten two light spheres in one frame is around 7 ms, a fifth of a
        -- frame at 30 fps but most of one at 120. Lower frame rate caps absorb
        -- spawn bursts more comfortably, not because the work shrinks but
        -- because there is more room for it.
        --
        -- IgnoreLightCurveExplanation
        --
        -- Some lights are timer lights: lanterns, campfires, and pals like
        -- Kitsunebi or FlameBambi. Palworld drives these from a day/night curve,
        -- so they light up in the evening, fade out around dawn, and stay dark
        -- all day. The curve is a multiplier on DefaultIntensity, which is why
        -- these entries set DefaultIntensity rather than Intensity: anything
        -- written to Intensity is overwritten a second later.
        --
        -- IgnoreLightCurve = true tells the light to ignore that curve, so it
        -- stays lit at DefaultIntensity around the clock. Set it to false, or
        -- remove the line, to get the normal day/night behaviour back.
        --
        -- This asks the game to ignore the curve rather than editing it, using
        -- the same mechanism Palworld uses internally, so it only affects the
        -- entries configured here and reverts cleanly. Turning off the light
        -- group, or reloading with the flag removed, restores the curve.
        --
        -- Only works on actors whose light is a timer light, meaning their
        -- NameContains includes ".BP_PalTimerPointLightComponent". It does
        -- nothing on any other entry and logs a warning if set there.
        --
        -- Note that a light with no DefaultIntensity configured will sit at
        -- whatever value the game gave it, which can be much brighter than
        -- expected once the curve is no longer dimming it.
        --
        -- INSTANCES
        --
        -- Instances are used when one actor needs multiple separately configured light
        -- components. Templates are resolved on the actor first, producing the actor's
        -- shared Settings. Every instance inherits those resolved actor Settings and
        -- then applies its own Settings as the final override.
        --
        -- UseExisting = true assigns one native component already owned by the actor.
        -- Actor-level NameContains filters the candidate pool first; optional instance
        -- NameContains applies an additional AND filter when selecting that component.
        -- Each existing component can be assigned to only one instance.
        --
        -- An enabled instance without UseExisting = true is created by UltraGraphics.
        -- The actor entry must resolve CreateIfMissing = true, normally by including
        -- the "CreatedPointLight" template. Created instance names are stable runtime
        -- slots, so pressing F8 updates them instead of creating duplicates.
        --
        -- Instances also support Enabled and Debug. Setting Enabled = false removes a
        -- previously created instance the next time the actor is processed.
        --
        -- FOLLOW
        --
        -- A created or existing instance can follow another resolved instance by Name:
        --
        --     Follow = {
        --         Source = "Primary",
        --         LightColor = true,
        --         Visibility = true,
        --     },
        --
        -- Only LightColor and Visibility are mirrored. Intensity, position, radius and
        -- all other properties remain controlled by the follower's own Settings. When
        -- the game lets players change a native lamp's color, leave LightColor unset on
        -- the actor/follower and use Follow.LightColor to preserve that in-game control.
        --
        -- Example: keep the native lamp and add a second synchronized point light:
        --
        --     Templates = {
        --         "PointLightDefaults",
        --         "CreatedPointLight",
        --     },
        --     Instances = {
        --         {
        --             Name = "Primary",
        --             UseExisting = true,
        --             Settings = {
        --                 RelativeLocation = { X = -50, Y = 0, Z = 320 },
        --             },
        --         },
        --         {
        --             Name = "Secondary",
        --             Follow = {
        --                 Source = "Primary",
        --                 LightColor = true,
        --                 Visibility = true,
        --             },
        --             Settings = {
        --                 RelativeLocation = { X = 50, Y = 0, Z = 320 },
        --             },
        --         },
        --     },
        --
        -- MESH SHADOW RULES
        --
        -- Meshes applies settings to StaticMeshComponents owned by the same actor:
        --
        --     Meshes = {
        --         {
        --             NameContains = { ".SM_LightPole_A5" },
        --             CastShadow = false,
        --         },
        --     },
        --
        -- Mesh NameContains is separate from the light-component NameContains above.
        -- Every listed substring must match. Omitting it targets every static mesh on
        -- the actor. CastShadow is component-wide: false prevents that mesh from
        -- casting shadows from ALL lights, not only the light configured in this entry.
        --
        -- Top-level actor fields such as ComponentClass, CreateIfMissing and
        -- DisableLightCulling apply to the complete actor entry and therefore affect
        -- all its instances. Template definitions and the Lights.Groups table are
        -- resolved before config.lua returns, leaving a flat actor-class table for
        -- main.lua.
        -- ============================================================
        Templates = {
            -- Base settings explicitly applied to point-light entries.
            PointLightDefaults = {
                ComponentClass = "PointLightComponent",
                Settings = {
                    CastShadows = true,
                    CastVolumetricShadow = true,
                    ShadowResolutionScale = 1.0,
                    ShadowBias = 0.3,
                    SourceRadius = 5,
                    --Intensity = 10.0, -- do not use intensity here or on any other defaults, will cause issues with stuff that has DefaultIntensity
                    IndirectLightingIntensity = 5,
                    VolumetricScatteringIntensity = 1,
                    LightFalloffExponent = 2.0,
                    AttenuationRadius = 1200.0,
                    IntensityUnits = 0,
                    InverseExposureBlend = 0,
                    bUseInverseSquaredFalloff = false,
                    bUseRayTracedDistanceFieldShadows = false,
                },
            },
            
            -- Shared defaults for PalSpheres
            PalSphereDefaults = {
                Settings = {
                    CastVolumetricShadow = true,
                    Intensity = 7,
                    SourceRadius = 5,
                    VolumetricScatteringIntensity = 0.8,
                    AttenuationRadius = 600.0,
                    LightFalloffExponent = 4.0,
                    IndirectLightingIntensity = 10.0,
                    RelativeLocation = { X = 0.0, Y = 0.0, Z = 16.0 },
                    bUseRayTracedDistanceFieldShadows = true,
                }
            },
            
            -- Shared defaults for thrown PalSpheres
            PalSphereThrownDefaults = {
                CreateEagerly = true, -- to find out what this does, search "CreateEagerlyExplanation", it is especially important to read "CreateEagerlyCost"
                Settings = {
                    Intensity = 14,
                    AttenuationRadius = 800.0,
                    RelativeLocation = { X = 0.0, Y = 0.0, Z = 0.0 },
                },
            },
            
            -- Shared defaults for Pal lights.
            PalDefaults = {
                Settings = {
                    SourceRadius = 50,
                    SoftSourceRadius = 100,
                    ShadowResolutionScale = 0.5,
                    IndirectLightingIntensity = 10.0,
                    VolumetricScatteringIntensity = 0,
                    LightFalloffExponent = 2.0,
                    AttenuationRadius = 800.0,
                },
            },
            
            -- Shared defaults for player built objects.
            BuildObjectDefaults = {
                DisableLightCulling = true,
                Settings = {
                    IndirectLightingIntensity = 10.0,
                    VolumetricScatteringIntensity = 0.5,
                    LightFalloffExponent = 2.0,
                    AttenuationRadius = 1200.0,
                },
            },
            
            -- Shared defaults for created Pal skill-effect lights.
            SkillEffectDefaults = {
                Settings = {
                    ShadowResolutionScale = 0.5,
                    IndirectLightingIntensity = 10.0,
                    VolumetricScatteringIntensity = 0.25,
                    LightFalloffExponent = 2.0,
                    AttenuationRadius = 1200.0,
                    RelativeLocation = { X = 0.0, Y = 0.0, Z = 0.0 },
                },
            },
            
            -- Shared defaults for weapon muzzle flashes.
            WeaponMuzzleDefaults = {
                -- Presence of this block is what turns the entry into a muzzle
                -- flash. Search "MuzzleFlashExplanation" for what the two
                -- numbers do and how to pick them.
                MuzzleFlash = {
                    -- Only turn on Fade for weapons that need a long flash,
                    -- some energy weapons for example, where holding full
                    -- intensity and then stopping dead looks wrong. Short
                    -- flashes do not need it. Lumen happens to fade the light
                    -- in and out by itself, but that is not something to rely
                    -- on: it needs surfaces nearby to bounce off, and players
                    -- with Lumen off get none of it. See
                    -- "MuzzleFlashExplanation".
                    Fade = false,
                    DurationMs = 35,
                    CoalesceMs = 25,
                },
                CreateEagerly = true, -- to find out what this does, search "CreateEagerlyExplanation", it is especially important to read "CreateEagerlyCost"
                Settings = {
                    Intensity = 10.0,
                    SourceRadius = 5,
                    AttenuationRadius = 2500.0,
                    LightFalloffExponent = 4.0,
                    IndirectLightingIntensity = 10.0,
                    VolumetricScatteringIntensity = 1.0,
                    LightColor = { R = 255, G = 200, B = 130, A = 255 },
                    -- If you experience performance issues, try using cheaper shadow or disable shadows
                    --CastShadows = false, -- disable shadows
                    --bUseRayTracedDistanceFieldShadows = true, -- cheaper Shadows
                },
            },
            
            -- Shared defaults for tornado skills
            SkillEffect_TornadoDefaults = {
                Settings = {
                    Intensity = 25,
                    VolumetricScatteringIntensity = 1.0,
                    AttenuationRadius = 2000,
                    RelativeLocation = { X = 0.0, Y = 0.0, Z = 200.0 }, -- middle of tornado
                },
            },
            
            -- Shared defaults for breath skills
            SkillEffect_BreathDefaults = {
                Settings = {
                    Intensity = 15,
                    VolumetricScatteringIntensity = 0.5,
                    AttenuationRadius = 2000.0,
                },
            },
            
            -- Shared defaults for created dungeon entrances and exits.
            DungeonPassageDefaults = {
                Settings = {
                    Intensity = 15.0,
                    AttenuationRadius = 2000.0,
                    SourceRadius = 50.0,
                    SoftSourceRadius = 100.0,
                    VolumetricScatteringIntensity = 1.0,
                    IndirectLightingIntensity = 1.5,
                },
            },
            
            -- Shared defaults for created dungeon "clutter" light sources.
            DungeonClutterDefaults = {
                Settings = {
                    MaxDrawDistance = 4000,
                    MaxDistanceFadeRange = 1000,
                    VolumetricScatteringIntensity = 0.75,
                    IndirectLightingIntensity = 10.0,
                    ShadowResolutionScale = 0.5,
                    AttenuationRadius = 800.0,
                    LightFalloffExponent = 2.0,
                },
            },
            
            -- Shared defaults for collectibles.
            CollectibleDefaults = {
                Settings = {
                    ShadowResolutionScale = 0.5,
                    IndirectLightingIntensity = 10,
                    AttenuationRadius = 800.0,
                },
            },
            
            -- Shared defaults for jump spots.
            JumpSpotDefaults = {
                Settings = {
                    Intensity = 2,
                    IndirectLightingIntensity = 2.0,
                    ShadowResolutionScale = 0.5,
                    LightColor = { R = 255, G = 255, B = 255, A = 255 },
                    RelativeLocation = { X = 0.0, Y = 0.0, Z = 200.0 }, -- center above
                },
            },
            
            -- Point-light component created by UltraGraphics.
            CreatedPointLight = {
                ComponentClass = "PointLightComponent",
                CreateIfMissing = true,
                Settings = {
                },
            },
            
            -- Disables skeletal meshes from casting shadow by using distance field shadows.
            NoSkeletalMeshShadow = {
                Settings = {
                    SourceRadius = 5, -- controls softness of distance field shadow.
                    bUseRayTracedDistanceFieldShadows = true,
                },
            },
            
            -- Common cyan-blue used by towers, dungeons and blue collectibles.
            LightColorCyan = {
                Settings = {
                    LightColor = { R = 0, G = 180, B = 255, A = 255 }, -- Cyan
                },
            },
            LightColorCyanPale = {
                Settings = {
                    LightColor = { R = 100, G = 230, B = 245, A = 255 }, -- Pale Cyan
                },
            },
            LightColorPurple = {
                Settings = {
                    LightColor = { R = 175, G = 105, B = 190, A = 255 },
                },
            },
            LightColorFire = {
                Settings = {
                    LightColor = { R = 255, G = 135, B = 10, A = 255 },
                    --LightColor = { R = 205, G = 105, B = 10, A = 255 }, -- orange
                    --LightColor = { R = 255, G = 135, B = 10, A = 255 }, -- orange
                },
            },
            LightColorFirePale = {
                Settings = {
                    LightColor = { R = 215, G = 150, B = 90, A = 255 },
                    --LightColor = { R = 215, G = 125, B = 65, A = 255 },
                },
            },
            LightColorLaserBlue = {
                Settings = {
                    LightColor = { R = 100, G = 200, B = 255, A = 255 }, -- laser Blue
                },
            },
            LightColorLaserRed = {
                Settings = {
                    LightColor = { R = 255, G = 120, B = 120, A = 255 }, -- laser Red
                },
            },
            LightColorPalSphere = {
                Settings = {
                    LightColor = { R = 50, G = 100, B = 185, A = 255 }, -- PalSphere, Blue
                },
            },
            LightColorPalSphereMega = {
                Settings = {
                    LightColor = { R = 50, G = 185, B = 100, A = 255 }, -- PalSphere Mega, Green
                },
            },
            LightColorPalSphereGiga = {
                Settings = {
                    LightColor = { R = 185, G = 165, B = 50, A = 255 }, -- PalSphere Giga, Yellow
                },
            },
            LightColorPalSphereHyper = {
                Settings = {
                    LightColor = { R = 185, G = 50, B = 50, A = 255 }, -- PalSphere Hyper, Red
                },
            },
            LightColorPalSphereUltra = {
                Settings = {
                    LightColor = { R = 185, G = 50, B = 140, A = 255 }, -- PalSphere Ultra, Pink
                },
            },
            LightColorPalSphereLegendary = {
                Settings = {
                    LightColor = { R = 100, G = 50, B = 185, A = 255 }, -- PalSphere Legendary, Purple
                },
            },
            -- Ultimate and Exotic have multiple light instances, so their colors are controlled through their corresponding light entries.
            LightColorPalSphereSol = {
                Settings = {
                    LightColor = { R = 185, G = 185, B = 185, A = 255 }, -- PalSphere Sol, Pale White
                },
            },
            LightColorPalSphereAncient = {
                Settings = {
                    LightColor = { R = 115, G = 175, B = 185, A = 255 }, -- PalSphere Ancient
                },
            },
        },
        
        -- ============================================================
        -- Environment
        -- ============================================================
        -- Sun
        BP_PalSkyCreator_C = {
            Enabled = true,
            Group = "Environment",
            ComponentClass = "DirectionalLightComponent",
            NameContains = {
                ".Sun Light Component"
            },
            -- LUMEN_NIGHT_TRANSITION_NOISE_FIX
            -- Lumen flickers badly while the sun is near the horizon, and the
            -- cause is the sun's own indirect lighting. These settings replace
            -- the ones below while it is dark, and the sun contributes no
            -- useful bounced light at night anyway, so the whole night is
            -- covered rather than only the two transitions.
            -- 
            -- The window comes from the sky's own SunriseTime and SunsetTime,
            -- so it follows the map rather than being hardcoded. Margin widens
            -- it at both ends, in game hours, because the check runs a few
            -- times a second rather than every frame: at higher day speed
            -- settings a fraction of a second is a larger slice of game time.
            -- Raise it if you still catch a flicker right at the edges.
            -- 
            -- Only the values listed here are replaced. Everything in Settings
            -- below, and anything weather mods do to Intensity, is untouched.
            -- Delete this block to keep the configured value all day.
            NightSettings = {
                Margin = 0.5,
                Settings = {
                    IndirectLightingIntensity = 0,
                },
            },
            Settings = {
                -- Regular Intensity is controlled by weather presets, check mod UltraWeather to adjust.
                IndirectLightingIntensity = 2, -- Vanilla: 1
                DistanceFieldShadowDistance = 100000.0 -- Vanilla: 25000.0
            }
        },
        
        -- ============================================================
        -- Player
        -- ============================================================
        -- Torch
        BP_Torch_C = {
            Enabled = true,
            DisableLightCulling = true,
            Group = "Player",
            Templates = {
                "PointLightDefaults",
                --"NoSkeletalMeshShadow", -- NoSelfShadow uncomment to disable torch from casting shadow on character
            },
            NameContains = {
                "BP_Torch_C_",
                ".PointLight"
            },
            Settings = {
                Intensity = 10.0, -- Vanilla: 20
                ShadowBias = 0.15, -- allow smaller objects to also cast shadow, like grass
                                   -- too low value causes banding issue for example on beeches
                AttenuationRadius = 2500.8, -- Vanilla: 5000
                --LightColor = { R = 231, G = 142, B = 31, A = 255 }, -- Vanilla: { R = 231, G = 142, B = 31, A = 255 }
            }
        },
        -- Lantern
        BP_Lamp_C = {
            Enabled = true,
            CreateEagerly = true, -- search "CreateEagerlyExplanation"
            --IgnoreLightCurve = true, -- search "IgnoreLightCurveExplanation"
            --SetDisableFlags = { IsInWorldTree = false }, -- Palworld switches these lamps off while you are inside world, uncomment to keep them lit there.
            DisableLightCulling = true,
            Group = "Player",
            Templates = {
                "PointLightDefaults",
                --"NoSkeletalMeshShadow", -- NoSelfShadow uncomment to disable lantern from casting shadow on character
            },
            NameContains = {
                "BP_Lamp_C_",
                ".BP_PalTimerPointLightComponent",
            },
            Settings = {
                DefaultIntensity = 10.0, -- Vanilla: 30
                ShadowBias = 0.15, -- allow smaller objects to also cast shadow, like grass
                VolumetricScatteringIntensity = 0,
                AttenuationRadius = 2000.0, -- Vanilla: 5000
                RelativeLocation = { X = 20.0, Y = 10.0, Z = 0.0 }, -- front to the side
            }
        },
        -- Enhanced Lantern
        BP_Lamp_High_C = {
            Enabled = true,
            CreateEagerly = true, -- search "CreateEagerlyExplanation"
            --IgnoreLightCurve = true, -- search "IgnoreLightCurveExplanation"
            --SetDisableFlags = { IsInWorldTree = false }, -- Palworld switches these lamps off while you are inside world, uncomment to keep them lit there.
            DisableLightCulling = true,
            Group = "Player",
            Templates = {
                "PointLightDefaults",
                --"NoSkeletalMeshShadow", -- NoSelfShadow uncomment to disable enhanced lantern from casting shadow on character
            },
            NameContains = {
                "BP_Lamp_High_C_",
                ".BP_PalTimerPointLightComponent",
            },
            Settings = {
                DefaultIntensity = 15.0, -- Vanilla: 50
                ShadowBias = 0.15, -- allow smaller objects to also cast shadow, like grass
                VolumetricScatteringIntensity = 0,
                AttenuationRadius = 4000.0, -- Vanilla: 10000
                RelativeLocation = { X = 20.0, Y = 10.0, Z = 0.0 }, -- front to the side
            }
        },
        
        -- ============================================================
        -- Weapons
        -- ============================================================
        -- Beeam Sword
        BP_BeamSword_C = {
            Enabled = true,
            Group = "Weapons",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "LightColorLaserBlue",
                --"NoSkeletalMeshShadow", -- NoSelfShadow uncomment to disable torch from casting shadow on character
            },
            Settings = {
                Intensity = 10,
                RelativeLocation = { X = 0.0, Y = 0.0, Z = 70.0 },
            }
        },
        -- Beeam Sword - Projectile
        BP_BeamSwordBullet_C = {
            Enabled = true,
            CreateEagerly = true,
            Group = "Weapons",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "LightColorLaserBlue",
            },
            Settings = {
                Intensity = 15,
                VolumetricScatteringIntensity = 0.5,
                AttenuationRadius = 2000,
                RelativeLocation = { X = 50.0, Y = 0.0, Z = 50.0 },
            }
        },
        -- Laser Sword
        BP_SkyBeamSword_C = {
            Enabled = true,
            Group = "Weapons",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "LightColorLaserRed",
                --"NoSkeletalMeshShadow", -- NoSelfShadow uncomment to disable torch from casting shadow on character
            },
            Settings = {
                Intensity = 10,
                RelativeLocation = { X = 0.0, Y = 0.0, Z = 70.0 },
            }
        },
        -- Laser Sword - Projectile
        BP_SkyBeamSwordBulle_C = { -- yes this is indeed the real actor class name
            Enabled = true,
            CreateEagerly = true,
            Group = "Weapons",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "LightColorLaserRed",
            },
            Settings = {
                Intensity = 15,
                VolumetricScatteringIntensity = 0.5,
                AttenuationRadius = 2000,
                RelativeLocation = { X = 50.0, Y = 0.0, Z = 50.0 },
            }
        },
        -- Rocket Launcher - Projectile
        BP_RocketBullet_C = {
            Enabled = true,
            CreateEagerly = true,
            Group = "Weapons",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "LightColorFire",
            },
            Settings = {
                Intensity = 25,
                VolumetricScatteringIntensity = 0.5,
                AttenuationRadius = 2500.0,
                RelativeLocation = { X = -120.0, Y = 0.0, Z = 0.0 }, -- behind
                --RelativeLocation = { X = 80.0, Y = 0.0, Z = 0.0 }, -- front
            }
        },
        -- Guided Missile Launcher - Projectile
        BP_HomingMissile_MissileLauncher_Big_C = {
            Enabled = true,
            CreateEagerly = true,
            Group = "Weapons",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "LightColorFire",
            },
            Settings = {
                Intensity = 25,
                VolumetricScatteringIntensity = 0.5,
                AttenuationRadius = 3500.0,
                RelativeLocation = { X = -90.0, Y = 0.0, Z = 0.0 }, -- behind
            }
        },
        -- ============================================================
        -- Weapons with Muzzle Flash
        -- ============================================================
        --
        -- MuzzleFlashExplanation
        --
        -- A weapon entry with a MuzzleFlash block gets one light, created when
        -- the weapon spawns as you equip it. The light sits at zero intensity
        -- and is only turned up for the few milliseconds of each shot, so
        -- firing costs one intensity write rather than creating a light per
        -- bullet.
        --
        --   DurationMs   how long the flash stays lit. Keep it below the
        --                weapon's time between shots or rapid fire turns into
        --                one continuous glow. The fastest weapon measured was
        --                the submachine gun at about 52 ms between shots.
        --
        --   CoalesceMs   shots arriving within this long after a flash starts
        --                are folded into it. Shotguns fire one bullet per
        --                pellet, nine of them inside a single frame, and this
        --                turns that into one flash. Must stay below the fire
        --                interval of the fastest weapon you use.
        --
        --   Fade         off by default. The mod's light is either on or off:
        --                it holds full intensity for DurationMs and then stops.
        --                Lumen normally hides that. Indirect light ramps in and
        --                out over several frames, so the flash appears to ease
        --                up and fade away on its own, even at durations of
        --                several seconds where no muzzle effect is left.
        --
        --                That smoothing needs two things: nearby surfaces for
        --                the light to bounce off, and enough
        --                IndirectLightingIntensity below to feed Lumen in the
        --                first place. Firing in the open, placing the light
        --                clear of geometry, or lowering that value all reduce
        --                it, and the raw cutoff becomes visible. It is also
        --                absent entirely for anyone playing with Lumen off.
        --
        --                Turn Fade on where the cutoff shows. The light then
        --                eases out itself: full brightness immediately, most of
        --                it gone by halfway, then a short tail. It costs an
        --                intensity write per frame while alive, so leave it off
        --                where Lumen is already doing the work.
        --
        -- MuzzleFlash is replaced as a whole, not merged, so an entry that
        -- defines its own block needs every value it wants, not just the one
        -- being changed. Copy the block from WeaponMuzzleDefaults and edit it.
        --
        -- RelativeLocation places the light at the muzzle. X is along the
        -- barrel, so it is roughly the barrel length in centimetres. The values
        -- below were read from each weapon's own muzzle socket, except where
        -- noted, and Y is 0 because the muzzle sits on the barrel axis.
        --
        -- Some weapons already own a light of their own, usually the glowing
        -- ones. For those the mod finds the existing light, writes the muzzle
        -- settings onto it and creates nothing, which leaves no light for the
        -- flash to drive: it ends up wherever the game put it, often buried
        -- inside the weapon, and never pulses. The tell is silence, the weapon
        -- produces no muzzle lines in the log at all while other weapons do.
        -- Wrapping the Settings in an Instances block fixes it, because a named
        -- instance is always created rather than adopted. BP_OverheatRifle_C
        -- and BP_BeamLauncher_C below are set up that way.
        --
        -- Nothing happens for a weapon without an entry here, so NPC and enemy
        -- weapons stay dark unless you add them yourself. Their class names end
        -- in _NPC_C, for example BP_AssaultRifle_NPC_C. Adding those is fine
        -- but a firefight can put several flashing lights on screen at once.
        --
        -- To correct an offset that looks wrong, set LogLevel.Lights = 2 and
        -- fire the weapon once. Each line names the actor class, which is also
        -- the entry name to edit here. The probe line prints a ready made
        -- RelativeLocation if the weapon exposes a muzzle socket, and says so
        -- when it does not, in which case adjust X until the light sits at the
        -- end of the barrel. The per shot lines print the gap between bullets,
        -- which is what CoalesceMs has to stay below, and mark the ones that
        -- were folded into the previous flash.
        --
        -- Adding a weapon that is not listed here needs an entry first: the log
        -- only covers weapons that already have a light, so copy one of the
        -- entries below, rename it to the weapon's actor class, fire once, and
        -- read the real offset from the log. The class name is visible in the
        -- UE4SS live view, or by watching the log while equipping.
        --
        -- The probe line appears once per weapon actor, so unequip and equip
        -- again to see it a second time.
        -- 
        -- Musket
        BP_Musket_C = {
            Enabled = true,
            Group = "Weapons",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "WeaponMuzzleDefaults",
            },
            Settings = {
                RelativeLocation = { X = 120.9, Y = -0.8, Z = 4.5 },
            }
        },
        -- Makeshift Handgun
        BP_MakeshiftHandgun_C = {
            Enabled = true,
            Group = "Weapons",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "WeaponMuzzleDefaults",
            },
            Settings = {
                RelativeLocation = { X = 35.1, Y = 0.0, Z = 14.7 },
            }
        },
        -- Makeshift SMG
        BP_MakeshiftSubmachineGun_C = {
            Enabled = true,
            Group = "Weapons",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "WeaponMuzzleDefaults",
            },
            Settings = {
                RelativeLocation = { X = 77.2, Y = 0.0, Z = 4.7 },
            }
        },
        -- Handgun
        BP_HandGun_C = {
            Enabled = true,
            Group = "Weapons",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "WeaponMuzzleDefaults",
            },
            Settings = {
                RelativeLocation = { X = 35.1, Y = 0.0, Z = 14.7 },
            }
        },
        -- Makeshift Shotgun
        BP_MakeshiftShotgun_C = {
            Enabled = true,
            Group = "Weapons",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "WeaponMuzzleDefaults",
            },
            Settings = {
                RelativeLocation = { X = 37.3, Y = 0.0, Z = 10.2 },
            }
        },
        -- Makeshift Assault Rifle
        BP_MakeshiftAssaultRifle_C = {
            Enabled = true,
            Group = "Weapons",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "WeaponMuzzleDefaults",
            },
            Settings = {
                RelativeLocation = { X = 76.9, Y = 0.0, Z = 8.8 },
            }
        },
        -- Old Revolver
        BP_OldRevolver_C = {
            Enabled = true,
            Group = "Weapons",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "WeaponMuzzleDefaults",
            },
            Settings = {
                RelativeLocation = { X = 33.2, Y = 0.0, Z = 12.9 },
            }
        },
        -- Single-Shot Rifle
        BP_SingleShotRifle_C = {
            Enabled = true,
            Group = "Weapons",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "WeaponMuzzleDefaults",
            },
            Settings = {
                RelativeLocation = { X = 80.8, Y = 0.0, Z = 7.6 },
            }
        },
        -- SMG
        BP_SubmachineGun_C = {
            Enabled = true,
            Group = "Weapons",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "WeaponMuzzleDefaults",
            },
            Settings = {
                RelativeLocation = { X = 29.5, Y = 0.0, Z = 11.5 },
            }
        },
        -- Doubled-Barreled Shotgun
        BP_DoubleBarrelShotgun_C = {
            Enabled = true,
            Group = "Weapons",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "WeaponMuzzleDefaults",
            },
            Settings = {
                RelativeLocation = { X = 115.0, Y = 0.0, Z = 8.0 },
            }
        },
        -- Semi-Auto Rifle
        BP_SemiAutoRifle_C = {
            Enabled = true,
            Group = "Weapons",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "WeaponMuzzleDefaults",
            },
            Settings = {
                RelativeLocation = { X = 102.5, Y = 0.0, Z = 5.3 },
            }
        },
        -- Pump-Action Shotgun
        BP_PumpActionShotgun_C = {
            Enabled = true,
            Group = "Weapons",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "WeaponMuzzleDefaults",
            },
            Settings = {
                RelativeLocation = { X = 78.0, Y = 0.0, Z = 8.0 },
            }
        },
        -- Assault Rifle
        BP_NormalRifle_C = {
            Enabled = true,
            Group = "Weapons",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "WeaponMuzzleDefaults",
            },
            Settings = {
                RelativeLocation = { X = 80.0, Y = 1.0, Z = 11.0 },
            }
        },
        -- Semi-Auto Shotgun
        BP_SemiAutoShotgun_C = {
            Enabled = true,
            Group = "Weapons",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "WeaponMuzzleDefaults",
            },
            Settings = {
                RelativeLocation = { X = 55.0, Y = 0.0, Z = 10.0 },
            }
        },
        -- Laser Rifle
        BP_LaserRifle_C = {
            Enabled = true,
            Group = "Weapons",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "WeaponMuzzleDefaults",
                "LightColorLaserBlue",
            },
            Settings = {
                RelativeLocation = { X = 56.5, Y = 0.0, Z = 16.6 },
            }
        },
        -- Plasma Multicutter
        BP_LaserMiningTool_C = {
            Enabled = false,
            Group = "Weapons",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "WeaponMuzzleDefaults",
                "LightColorLaserBlue",
            },
            Settings = {
                RelativeLocation = { X = 110.0, Y = 0.0, Z = -30.0 },
            }
        },
        -- Gatling Gun
        BP_GatlingGun_C = {
            Enabled = true,
            Group = "Weapons",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "WeaponMuzzleDefaults",
            },
            Settings = {
                RelativeLocation = { X = 144.0, Y = 0.0, Z = -20.0 },
            }
        },
        -----------------------------------------------------------------------------------
        -- TODO: SORT THE BELOW, THEY ARE UNSORTED
        -----------------------------------------------------------------------------------
        -- Vortex Beater
        BP_YakushimaGun001_C = {
            Enabled = true,
            Group = "Weapons",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "WeaponMuzzleDefaults",
                --"LightColorLaserBlue",
            },
            Settings = {
                RelativeLocation = { X = 80.0, Y = 0.0, Z = 10.0 },
            }
        },
        -- Charge Rifle
        BP_ChargeLaserRifle_C = {
            Enabled = true,
            Group = "Weapons",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "WeaponMuzzleDefaults",
                "LightColorLaserRed",
            },
            MuzzleFlash = {
                Fade = true,
                DurationMs = 700,
                CoalesceMs = 25,
            },
            Settings = {
                RelativeLocation = { X = 102.0, Y = 0.0, Z = 3.0 },
            }
        },
        -- Charge Rifle - Projectile, TODO: it lingers in world, need to fix before uncommenting
        --BP_ChargeLaserRifleBullet_C = {
        --    Enabled = true,
        --    CreateEagerly = true,
        --    Group = "Weapons",
        --    Templates = {
        --        "PointLightDefaults",
        --        "CreatedPointLight",
        --        "LightColorLaserRed",
        --    },
        --    Settings = {
        --        Intensity = 0,
        --        AttenuationRadius = 1500,
        --        RelativeLocation = { X = 0.0, Y = 0.0, Z = 0.0 },
        --    }
        --},
        -- Plasma Rifle
        BP_ElectricArcAssaultRifle_C = {
            Enabled = true,
            Group = "Weapons",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "WeaponMuzzleDefaults",
                "LightColorLaserBlue",
            },
            Settings = {
                RelativeLocation = { X = 105.0, Y = 0.0, Z = 9.0 },
            }
        },
        -- Overheat Rifle
        -- Owns a native light, so this creates its own instead of adopting it.
        -- Search "MuzzleFlashExplanation" for why that matters.
        BP_OverheatRifle_C = {
            Enabled = true,
            Group = "Weapons",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "WeaponMuzzleDefaults",
                "LightColorLaserRed",
            },
            Instances = {
                {
                    Name = "Muzzle",
                    Settings = {
                        RelativeLocation = { X = 83.0, Y = 0.0, Z = 10.9 },
                    },
                },
            },
        },
        -- Beam Launcher
        -- Owns a native light, so this creates its own instead of adopting it.
        -- Search "MuzzleFlashExplanation" for why that matters.
        BP_BeamLauncher_C = {
            Enabled = true,
            Group = "Weapons",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "WeaponMuzzleDefaults",
                "LightColorLaserBlue",
            },
            MuzzleFlash = {
                Fade = true,
                DurationMs = 300,
                CoalesceMs = 25,
            },
            Instances = {
                {
                    Name = "Muzzle",
                    Settings = {
                        RelativeLocation = { X = 69.1, Y = 0.0, Z = 22.0 },
                    },
                },
            },
        },
        -- Beam Scatter
        BP_WidePenetrateShotgun_C = {
            Enabled = true,
            Group = "Weapons",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "WeaponMuzzleDefaults",
                "LightColorLaserBlue",
            },
            Settings = {
                RelativeLocation = { X = 82.1, Y = 0.0, Z = 12.0 },
            }
        },
        -- Combat SMG
        BP_SkySubmachineGun_C = {
            Enabled = true,
            Group = "Weapons",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "WeaponMuzzleDefaults",
            },
            Settings = {
                RelativeLocation = { X = 30.6, Y = 0.0, Z = 9.3 },
            }
        },
        -- Heavy Assault Rifle
        BP_SkyAssaultRifle_C = {
            Enabled = true,
            Group = "Weapons",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "WeaponMuzzleDefaults",
            },
            Settings = {
                RelativeLocation = { X = 34.7, Y = 0.0, Z = 8.4 },
            }
        },
        -- Energy Shotgun
        BP_EnergyShotgun_C = {
            Enabled = true,
            Group = "Weapons",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "WeaponMuzzleDefaults",
                "LightColorLaserBlue",
            },
            Settings = {
                RelativeLocation = { X = 75.0, Y = 0.0, Z = 10.0 },
            }
        },
        -- Laser Gatling Gun
        BP_LaserGatlingGun_C = {
            Enabled = true,
            Group = "Weapons",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "WeaponMuzzleDefaults",
                "LightColorLaserBlue",
            },
            Settings = {
                RelativeLocation = { X = 133.0, Y = 0.0, Z = -23.0 },
            }
        },
        ---- Untested
        --BP_OctaviaShotgun001_C = {
        --    Enabled = true,
        --    Group = "Weapons",
        --    Templates = {
        --        "PointLightDefaults",
        --        "CreatedPointLight",
        --        "WeaponMuzzleDefaults",
        --    },
        --    Settings = {
        --        RelativeLocation = { X = 60.0, Y = 0.0, Z = 8.0 },
        --    }
        --},
        ---- Untested
        ---- Marksman Revolver
        --BP_OctaviaRevolver001_C = {
        --    Enabled = true,
        --    Group = "Weapons",
        --    Templates = {
        --        "PointLightDefaults",
        --        "CreatedPointLight",
        --        "WeaponMuzzleDefaults",
        --    },
        --    Settings = {
        --        RelativeLocation = { X = 50.0, Y = 0.0, Z = 10.0 },
        --    }
        --},
        
        -- ============================================================
        -- Pal Spheres
        -- ============================================================
        -- BP_PalSphere_C             = When held
        -- BP_Item_PalSphere_C        = On conveyor belt
        -- BP_PalSphere_ThrowObject_C = When thrown
        -- 
        -- PalSphere
        BP_PalSphere_C = {
            Enabled = true,
            Group = "PalSpheres",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "PalSphereDefaults",
                "LightColorPalSphere",
            },
        },
        BP_Item_PalSphere_C = {
            Enabled = true,
            Group = "PalSpheres",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "PalSphereDefaults",
                "LightColorPalSphere",
            },
        },
        BP_PalSphere_ThrowObject_C = {
            Enabled = true,
            Group = "PalSpheres",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "PalSphereDefaults",
                "PalSphereThrownDefaults",
                "LightColorPalSphere",
            },
        },
        -- PalSphere Mega
        BP_PalSphere_Mega_C = {
            Enabled = true,
            Group = "PalSpheres",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "PalSphereDefaults",
                "LightColorPalSphereMega",
            },
        },
        BP_Item_PalSphereMega_C = {
            Enabled = true,       
            Group = "PalSpheres",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "PalSphereDefaults",
                "LightColorPalSphereMega",
            },
        },
        BP_PalSphere_ThrowObject_Mega_C = {
            Enabled = true,
            Group = "PalSpheres",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "PalSphereDefaults",
                "PalSphereThrownDefaults",
                "LightColorPalSphereMega",
            },
        },
        -- PalSphere Giga
        BP_PalSphere_Giga_C = {
            Enabled = true,
            Group = "PalSpheres",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "PalSphereDefaults",
                "LightColorPalSphereGiga",
            },
        },
        BP_Item_PalSphereGiga_C = {
            Enabled = true,            
            Group = "PalSpheres",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "PalSphereDefaults",
                "LightColorPalSphereGiga",
            },
        },
        BP_PalSphere_ThrowObject_Giga_C = {
            Enabled = true,
            Group = "PalSpheres",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "PalSphereDefaults",
                "PalSphereThrownDefaults",
                "LightColorPalSphereGiga",
            },
        },
        -- PalSphere Hyper
        BP_PalSphere_Tera_C = {
            Enabled = true,
            Group = "PalSpheres",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "PalSphereDefaults",
                "LightColorPalSphereHyper",
            },
        },
        BP_Item_PalSphereTera_C = {
            Enabled = true,            
            Group = "PalSpheres",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "PalSphereDefaults",
                "LightColorPalSphereHyper",
            },
        },
        BP_PalSphere_ThrowObject_Tera_C = {
            Enabled = true,
            Group = "PalSpheres",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "PalSphereDefaults",
                "PalSphereThrownDefaults",
                "LightColorPalSphereHyper",
            },
        },
        -- PalSphere Ultra
        BP_PalSphere_Master_C = {
            Enabled = true,
            Group = "PalSpheres",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "PalSphereDefaults",
                "LightColorPalSphereUltra",
            },
        },
        BP_Item_PalSphere_Master_C = {
            Enabled = true,
            Group = "PalSpheres",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "PalSphereDefaults",
                "LightColorPalSphereUltra",
            },
        },
        BP_PalSphere_ThrowObject_Master_C = {
            Enabled = true,
            Group = "PalSpheres",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "PalSphereDefaults",
                "PalSphereThrownDefaults",
                "LightColorPalSphereUltra",
            },
        },
        -- PalSphere Legendary
        BP_PalSphere_Legend_C = {
            Enabled = true,
            Group = "PalSpheres",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "PalSphereDefaults",
                "LightColorPalSphereLegendary",
            },
        },
        BP_Item_PalSphere_Legend_C = {
            Enabled = true,
            Group = "PalSpheres",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "PalSphereDefaults",
                "LightColorPalSphereLegendary",
            },
        },
        BP_PalSphere_ThrowObject_Legend_C = {
            Enabled = true,
            Group = "PalSpheres",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "PalSphereDefaults",
                "PalSphereThrownDefaults",
                "LightColorPalSphereLegendary",
            },
        },
        -- PalSphere Ultimate
        BP_PalSphere_Ultimate_C = {
            Enabled = true,
            Group = "PalSpheres",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "PalSphereDefaults",
            },
            Instances = {
                {
                    Name = "Light1",
                    Settings = {
                        RelativeLocation = { X = 10.0, Y = 0.0, Z = 0.0 },
                        LightColor = { R = 0, G = 60, B = 150, A = 255 }, -- Blue
                    },
                },
                {
                    Name = "Light2",
                    Settings = {
                        RelativeLocation = { X = -10.0, Y = 0.0, Z = 0.0 },
                        LightColor = { R = 130, G = 30, B = 150, A = 255 }, -- Purple
                    },
                },
            },
        },
        BP_Item_PalSphere_Ultimate_C = {
            Enabled = true,
            Group = "PalSpheres",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "PalSphereDefaults",
            },
            Instances = {
                {
                    Name = "Light1",
                    Settings = {
                        RelativeLocation = { X = 16.0, Y = 0.0, Z = 16.0 },
                        LightColor = { R = 50, G = 100, B = 185, A = 255 }, -- Blue
                    },
                },
                {
                    Name = "Light2",
                    Settings = {
                        RelativeLocation = { X = -16.0, Y = 0.0, Z = 16.0 },
                        LightColor = { R = 185, G = 50, B = 180, A = 255 }, -- Pink
                    },
                },
            },
        },
        BP_PalSphere_ThrowObject_Ultimate_C = {
            Enabled = true,
            Group = "PalSpheres",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "PalSphereDefaults",
                "PalSphereThrownDefaults",
            },
            Instances = {
                {
                    Name = "Light1",
                    Settings = {
                        RelativeLocation = { X = 35.0, Y = 0.0, Z = 0.0 },
                        LightColor = { R = 0, G = 60, B = 150, A = 255 }, -- Blue
                    },
                },
                {
                    Name = "Light2",
                    Settings = {
                        RelativeLocation = { X = -35.0, Y = 0.0, Z = 0.0 },
                        LightColor = { R = 130, G = 30, B = 150, A = 255 }, -- Purple
                    },
                },
            },
        },
        -- PalSphere Exotic
        BP_PalSphere_Exotic_C = {
            Enabled = true,
            Group = "PalSpheres",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "PalSphereDefaults",
            },
            Instances = {
                {
                    Name = "Light1",
                    Settings = {
                        RelativeLocation = { X = 16.0, Y = 0.0, Z = 0.0 },
                        LightColor = { R = 150, G = 40, B = 0, A = 255 }, -- Exotic Red
                    },
                },
                {
                    Name = "Light2",
                    Settings = {
                        RelativeLocation = { X = -8.0, Y = 13.86, Z = 0.0 },
                        LightColor = { R = 0, G = 150, B = 40, A = 255 }, -- Exotic Green
                    },
                },
                {
                    Name = "Light3",
                    Settings = {
                        RelativeLocation = { X = -8.0, Y = -13.86, Z = 0.0 },
                        LightColor = { R = 40, G = 0, B = 150, A = 255 }, -- Exotic Blue
                    },
                },
            },
        },
        BP_Item_PalSphere_Exotic_C = {
            Enabled = true,
            Group = "PalSpheres",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "PalSphereDefaults",
            },
            Instances = {
                {
                    Name = "Light1",
                    Settings = {
                        RelativeLocation = { X = 16.0, Y = 0.0, Z = 16.0 },
                        LightColor = { R = 150, G = 40, B = 0, A = 255 }, -- Exotic Red
                    },
                },
                {
                    Name = "Light2",
                    Settings = {
                        RelativeLocation = { X = -8.0, Y = 13.86, Z = 16.0 },
                        LightColor = { R = 0, G = 150, B = 40, A = 255 }, -- Exotic Green
                    },
                },
                {
                    Name = "Light3",
                    Settings = {
                        RelativeLocation = { X = -8.0, Y = -13.86, Z = 16.0 },
                        LightColor = { R = 40, G = 0, B = 150, A = 255 }, -- Exotic Blue
                    },
                },
            },
        },
        BP_PalSphere_ThrowObject_Exotic_C = {
            Enabled = true,
            Group = "PalSpheres",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "PalSphereDefaults",
                "PalSphereThrownDefaults",
            },
            Instances = {
                {
                    Name = "Light1",
                    Settings = {
                        RelativeLocation = { X = 35.0, Y = 0.0, Z = 0.0 },
                        LightColor = { R = 185, G = 50, B = 0, A = 255 }, -- Exotic Red
                    },
                },
                {
                    Name = "Light2",
                    Settings = {
                        RelativeLocation = { X = -20.0, Y = 30, Z = 0.0 },
                        LightColor = { R = 0, G = 185, B = 50, A = 255 }, -- Exotic Green
                    },
                },
                {
                    Name = "Light3",
                    Settings = {
                        RelativeLocation = { X = -20.0, Y = -30.0, Z = 0.0 },
                        LightColor = { R = 50, G = 0, B = 185, A = 255 }, -- Exotic Blue
                    },
                },
            },
        },
        -- PalSphere Sol
        BP_PalSphere_Ancient1_C = {
            Enabled = true,
            Group = "PalSpheres",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "PalSphereDefaults",
                "LightColorPalSphereSol",
            },
        },
        BP_Item_PalSphere_Ancient1_C = {
            Enabled = true,
            Group = "PalSpheres",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "PalSphereDefaults",
                "LightColorPalSphereSol",
            },
        },
        BP_PalSphere_ThrowObject_Ancient1_C = {
            Enabled = true,
            Group = "PalSpheres",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "PalSphereDefaults",
                "PalSphereThrownDefaults",
                "LightColorPalSphereSol",
            },
        },
        -- PalSphere Ancient
        BP_PalSphere_Ancient2_C = {
            Enabled = true,
            Group = "PalSpheres",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "PalSphereDefaults",
                "LightColorPalSphereAncient",
            },
        },
        BP_Item_PalSphere_Ancient2_C = {
            Enabled = true,
            Group = "PalSpheres",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "PalSphereDefaults",
                "LightColorPalSphereAncient",
            },
        },
        BP_PalSphere_ThrowObject_Ancient2_C = {
            Enabled = true,
            Group = "PalSpheres",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "PalSphereDefaults",
                "PalSphereThrownDefaults",
                "LightColorPalSphereAncient",
            },
        },
        
        -- ============================================================
        -- Pals
        -- ============================================================
        -- Foxparks
        BP_Kitsunebi_C = {
            Enabled = true,
            IgnoreLightCurve = true, -- search "IgnoreLightCurveExplanation"
            Group = "Pals",
            Templates = {
                "PointLightDefaults",
                "PalDefaults",
            },
            NameContains = {
                "BP_Kitsunebi_C_",
                ".BP_PalTimerPointLightComponent"
            },
            Settings = {
                DefaultIntensity = 30.0,
                RelativeLocation = { X = 0.0, Y = 0.0, Z = 150.0 }, -- center above -- Vanilla: { X = 0.0, Y = 0.0, Z = 90.0 }
                --RelativeLocation = { X = 0.0, Y = -57.0, Z = 70.0 }, -- tail
                --RelativeLocation = { X = 0.0, Y = 70.0, Z = 70.0 }, -- front head
            }
        },
        -- Rooby
        BP_FlameBambi_C = {
            Enabled = true,
            IgnoreLightCurve = true, -- search "IgnoreLightCurveExplanation"
            Group = "Pals",
            Templates = {
                "PointLightDefaults",
                "PalDefaults",
            },
            NameContains = {
                "BP_FlameBambi_C_",
                ".BP_PalTimerPointLightComponent"
            },
            Settings = {
                DefaultIntensity = 30.0,
                RelativeLocation = { X = 0.0, Y = 30.0, Z = 170.0 }, -- center above
            }
        },
        -- Helzephyr
        BP_HadesBird_C = {
            Enabled = true,
            IgnoreLightCurve = true, -- search "IgnoreLightCurveExplanation"
            Group = "Pals",
            Templates = {
                "PointLightDefaults",
                "PalDefaults",
            },
            NameContains = {
                "BP_HadesBird_C_",
                ".BP_PalTimerPointLightComponent"
            },
            Settings = {
                VolumetricScatteringIntensity = 0.25,
                DefaultIntensity = 20.0, -- Vanilla: 20
                AttenuationRadius = 1000.0,
                LightColor = { R = 130.0, G = 70.0, B = 130.0 }, -- Vanilla: { R = 129.0, G = 68.0, B = 129.0 }
                RelativeLocation = { X = 0.0, Y = 70.0, Z = 350.0 }, -- head -- Vanilla: { X = 0.0, Y = 0.0, Z = 570.0 }
            }
        },
        -- Helzephyr Lux
        BP_HadesBird_Electric_C = {
            Enabled = true,
            IgnoreLightCurve = true, -- search "IgnoreLightCurveExplanation"
            Group = "Pals",
            Templates = {
                "PointLightDefaults",
                "PalDefaults",
            },
            NameContains = {
                "BP_HadesBird_Electric_C_",
                ".BP_PalTimerPointLightComponent"
            },
            Settings = {
                VolumetricScatteringIntensity = 0.25,
                DefaultIntensity = 20.0, -- Vanilla: 20
                AttenuationRadius = 1000.0,
                LightColor = { R = 130.0, G = 130.0, B = 70.0 }, -- Vanilla: { R = 255.0, G = 218.0, B = 3.0 }
                RelativeLocation = { X = 0.0, Y = 70.0, Z = 350.0 }, -- head -- Vanilla: { X = 0.0, Y = 0.0, Z = 570.0 }
            }
        },
        -- Tombat
        BP_CatBat_C = {
            Enabled = true,
            IgnoreLightCurve = true, -- search "IgnoreLightCurveExplanation"
            Group = "Pals",
            Templates = {
                "PointLightDefaults",
                "PalDefaults",
            },
            NameContains = {
                "BP_CatBat_C_",
                ".BP_PalTimerPointLightComponent"
            },
            Settings = {
                DefaultIntensity = 20.0,
                LightColor = { R = 130, G = 100, B = 255, A = 255 }, -- purple
                RelativeLocation = { X = 0.0, Y = 20.0, Z = 300.0 }, -- top then a little behind
            }
        },
        -- Orserk
        BP_ThunderDragonMan_C = {
            Enabled = true,
            IgnoreLightCurve = true, -- search "IgnoreLightCurveExplanation"
            Group = "Pals",
            Templates = {
                "PointLightDefaults",
                "PalDefaults",
            },
            NameContains = {
                "BP_ThunderDragonMan_C_",
                ".BP_PalTimerPointLightComponent"
            },
            Settings = {
                DefaultIntensity = 20.0,
                RelativeLocation = { X = 0.0, Y = -50.0, Z = 170.0 }, -- behind between wings
            }
        },
        -- Jetragon
        BP_JetDragon_C = {
            Enabled = true,
            Group = "Pals",
            Templates = {
                "PointLightDefaults",
                "PalDefaults",
                "CreatedPointLight",
            },
            Settings = {
                Intensity = 10.0,
                AttenuationRadius = 1500.0,
                LightColor = { R = 255, G = 130, B = 125, A = 255 }, -- pink/red
                RelativeLocation = { X = -200.0, Y = 0.0, Z = 300.0 }, -- center between tail and wings
            }
        },
        -- ============================================================
        -- PredatorPals
        -- ============================================================
        BP_FairyDragon_BOSS_Predator_C = {
            Enabled = true,
            Group = "Pals",
            Templates = {
                "PointLightDefaults",
                "PalDefaults",
                "CreatedPointLight",
            },
            Settings = {
                Intensity = 30.0,
                VolumetricScatteringIntensity = 0.25,
                AttenuationRadius = 4000.0,
                LightColor = { R = 235, G = 25, B = 105, A = 255 }, -- predator pink
                RelativeLocation = { X = 0.0, Y = 0.0, Z = 500.0 }, -- top then a little behind
            }
        },
        -- Sootseer
        BP_CandleGhost_BOSS_Predator_C = {
            Enabled = true,
            Group = "Pals",
            Templates = {
                "PointLightDefaults",
                "PalDefaults",
                "CreatedPointLight",
            },
            Settings = {
                Intensity = 30.0,
                VolumetricScatteringIntensity = 0.25,
                AttenuationRadius = 4000.0,
                LightColor = { R = 235, G = 25, B = 105, A = 255 }, -- predator pink
                RelativeLocation = { X = 0.0, Y = 0.0, Z = 500.0 }, -- top then a little behind
            }
        },
        
        -- ============================================================
        -- Build Objects
        -- ============================================================
        -- Campfire
        BP_BuildObject_CampFire_C = {
            Enabled = true,
            Group = "BuildObjects",
            Templates = {
                "PointLightDefaults",
                "BuildObjectDefaults",
            },
            NameContains = {
                "BP_BuildObject_CampFire_C_",
                ".BP_PalFirePointLight"
            },
            Settings = {
                VolumetricScatteringIntensity = 0.1,
                ShadowBias = 0.75,
                RelativeLocation = { X = 0.0, Y = 0.0, Z = 120.0 }, -- Vanilla: { X = 0.0, Y = 0.0, Z = 151.6 },
            }
        },
        -- Mounted Torch
        BP_BuildObject_TorchStand_C = {
            Enabled = true,
            Group = "BuildObjects",
            Templates = {
                "PointLightDefaults",
                "BuildObjectDefaults",
            },
            NameContains = {
                "BP_BuildObject_TorchStand_C_",
                ".PointLight"
            },
            Settings = {
                Intensity = 10.0, -- Vanilla: 20
                RelativeLocation = { X = 1.0, Y = -3.0, Z = 130.0 },
            }
        },
        -- Brick Fireplace
        BP_BuildObject_Light_FirePlace01_C = {
            Enabled = true,
            Group = "BuildObjects",
            Templates = {
                "PointLightDefaults",
                "BuildObjectDefaults",
            },
            NameContains = {
                "BP_BuildObject_Light_FirePlace01_C_",
                ".PointLight"
            },
            Settings = {
                Intensity = 12.0, -- Vanilla: 20
                VolumetricScatteringIntensity = 1.5,
                RelativeLocation = { X = -3.0, Y = 1.0, Z = 32.0 } -- Vanilla: { X = 0.0, Y = -8.2, Z = 33.0 }
            }
        },
        -- Wall Torch
        BP_BuildObject_TorchHang_C = {
            Enabled = true,
            Group = "BuildObjects",
            Templates = {
                "PointLightDefaults",
                "BuildObjectDefaults",
            },
            NameContains = {
                "BP_BuildObject_TorchHang_C_",
                ".PointLight"
            },
            Settings = {
                Intensity = 10.0, -- Vanilla: 20
                --LightColor = { R = 231, G = 142, B = 31, A = 255 }, -- Vanilla: { R = 231, G = 142, B = 31, A = 255 }
                RelativeLocation = { X = 30.0, Y = 0.0, Z = 60.0 }, -- Vanilla: { X = 20.0, Y = 0.0, Z = 50.0 },
            }
        },
        -- Wall Lamp
        BP_BuildObject_Light_CandleSticks_Wall_C = {
            Enabled = true,
            Group = "BuildObjects",
            Templates = {
                "PointLightDefaults",
                "BuildObjectDefaults",
            },
            NameContains = {
                "BP_BuildObject_Light_CandleSticks_Wall_C_",
                ".PointLight"
            },
            Settings = {
                Intensity = 10.0, -- Vanilla: 20
                --LightColor = { R = 231, G = 211, B = 149, A = 255 }, -- do not override here, this light allows color to be set in game.
                AttenuationRadius = 700.0,
                RelativeLocation = { X = 50.0, Y = 0.0, Z = 40.0 },
            }
        },
        -- Chandelier
        BP_BuildObject_Light_CandleSticks_Top_C = {
            Enabled = true,
            Group = "BuildObjects",
            Templates = {
                "PointLightDefaults",
                "BuildObjectDefaults",
                "CreatedPointLight",
            },
            NameContains = {
                "BP_BuildObject_Light_CandleSticks_Top_C_",
                ".PointLight"
            },
            Instances = {
                {
                    Name = "Primary",
                    UseExisting = true,
                    Settings = {
                        Intensity = 10.0, -- Vanilla: 20
                        AttenuationRadius = 700.0,
                        RelativeLocation = { X = 0.0, Y = 0.0, Z = -140.0 },
                    },
                },
                {
                    Name = "SecondaryIntense",
                    Follow = {
                        Source = "Primary",
                        LightColor = true,
                        Visibility = true,
                    },
                    Settings = {
                        CastShadows = false,
                        Intensity = 400.0,
                        SourceRadius = 5,
                        SoftSourceRadius = 10,
                        AttenuationRadius = 50.0,
                        RelativeLocation = { X = 0.0, Y = 0.0, Z = -100.0 },
                    },
                },
            },
            Settings = {
                --LightColor = { R = 231, G = 183, B = 146, A = 255 }, -- do not override here, this light allows color to be set in game.
            }
        },
        -- Lamp
        BP_BuildObject_LampStand_C = {
            Enabled = true,
            Group = "BuildObjects",
            Templates = {
                "PointLightDefaults",
                "BuildObjectDefaults",
            },
            NameContains = {
                "BP_BuildObject_LampStand_C_",
                ".PointLight"
            },
            Settings = {
                Intensity = 12.5, -- Vanilla: 30
                --LightColor = { R = 231, G = 211, B = 149, A = 255 }, -- do not override here, this light allows color to be set in game.
                RelativeLocation = { X = 0.0, Y = 0.0, Z = 160.0 },
            }
        },
        -- Antique Brown Floor Lamp
        BP_BuildObject_Light_FloorLamp01_C = {
            Enabled = true,
            Group = "BuildObjects",
            Templates = {
                "PointLightDefaults",
                "BuildObjectDefaults",
                "CreatedPointLight",
            },
            NameContains = {
                "BP_BuildObject_Light_FloorLamp01_C_",
                ".PointLight"
            },
            Instances = {
                {
                    Name = "Primary",
                    UseExisting = true,
                    Settings = {
                        RelativeLocation = { X = 0.0, Y = 0.0, Z = 182.0 },
                    },
                },
                {
                    -- Secondary light just to brighten the hood itself
                    Name = "Secondary",
                    Follow = {
                        Source = "Primary",
                        LightColor = true,
                        Visibility = true,
                    },
                    Settings = {
                        Intensity = 100.0,
                        SourceRadius = 5,
                        AttenuationRadius = 110.0,
                        RelativeLocation = { X = 0.0, Y = 0.0, Z = 195.0 },
                    },
                },
            },
            Settings = {
                Intensity = 10.0, -- Vanilla: 20
                AttenuationRadius = 800.0,
                LightFalloffExponent = 4.0,
                ShadowResolutionScale = 0.5,
                --LightColor = { R = 231, G = 179, B = 144, A = 255 }, -- do not override here, this light allows color to be set in game.
            }
        },
        -- Antique Red Floor Lamp
        BP_BuildObject_Light_FloorLamp02_C = {
            Enabled = true,
            Group = "BuildObjects",
            Templates = {
                "PointLightDefaults",
                "BuildObjectDefaults",
                "CreatedPointLight",
            },
            NameContains = {
                "BP_BuildObject_Light_FloorLamp02_C_",
                ".PointLight"
            },
            Instances = {
                {
                    Name = "Primary",
                    UseExisting = true,
                    Settings = {
                        RelativeLocation = { X = 0.0, Y = 0.0, Z = 150.0 },
                    },
                },
                {
                    -- Secondary light just to brighten the hood itself
                    Name = "Secondary",
                    Follow = {
                        Source = "Primary",
                        LightColor = true,
                        Visibility = true,
                    },
                    Settings = {
                        Intensity = 200.0,
                        SourceRadius = 5,
                        AttenuationRadius = 110.0,
                        RelativeLocation = { X = 0.0, Y = 0.0, Z = 155.0 },
                    },
                },
            },
            Settings = {
                Intensity = 10.0, -- Vanilla: 20
                AttenuationRadius = 1000.0,
                LightFalloffExponent = 4.0,
                ShadowResolutionScale = 0.15, -- lower res in order to get softer shadow
                --LightColor = { R = 231, G = 179, B = 144, A = 255 }, -- do not override here, this light allows color to be set in game.
                
            }
        },
        -- Large Mounted Lamp
        BP_BuildObject_LampStandLarge_C = {
            Enabled = true,
            Group = "BuildObjects",
            Templates = {
                "PointLightDefaults",
                "BuildObjectDefaults",
            },
            NameContains = {
                "BP_BuildObject_LampStandLarge_C_",
                ".PointLight"
            },
            Settings = {
                Intensity = 20.0, -- Vanilla: 40
                IndirectLightingIntensity = 25,
                AttenuationRadius = 2500.0, -- Vanilla: 5000
                --LightColor -- do not override here, can be set in game.
                --LightColor = { R = 200, G = 230, B = 230, A = 255 }, -- TODO: update the GEN_VARIABLE template so we can set the default color of lamps but still allow them to be changed in game, default color should be this light blue which is common in work lights
                RelativeLocation = { X = 0.0, Y = 0.0, Z = 240.0 }, -- Above
                --RelativeLocation = { X = 0.0, Y = 0.0, Z = 140.0 }, -- Middle, ugly shadows, broken volumetric fog
            }
        },
        -- Large Ceiling Lamp
        BP_BuildObject_LampTopLarge_C = {
            Enabled = true,
            Group = "BuildObjects",
            Templates = {
                "PointLightDefaults",
                "BuildObjectDefaults",
            },
            NameContains = {
                "BP_BuildObject_LampTopLarge_C_",
                ".PointLight"
            },
            Settings = {
                Intensity = 10.0, -- Vanilla: 40
                AttenuationRadius = 1500.0, -- Vanilla: 3500
                RelativeLocation = { X = 0.0, Y = 0.0, Z = -35.0 },
            }
        },
        -- Ceiling Lamp
        BP_BuildObject_LampTop_C = {
            Enabled = true,
            Group = "BuildObjects",
            Templates = {
                "PointLightDefaults",
                "BuildObjectDefaults",
            },
            NameContains = {
                "BP_BuildObject_LampTop_C_",
                ".PointLight"
            },
            Settings = {
                Intensity = 10.0, -- Vanilla: 30
                RelativeLocation = { X = 0.0, Y = 0.0, Z = -50.0 },
            }
        },
        -- Japanese-Style Door A (Lamp top left)
        BP_BuildObject_JapaneseStyle_DoorWall_01_C = {
            Enabled = true,
            Group = "BuildObjects",
            Templates = {
                "PointLightDefaults",
                "BuildObjectDefaults",
                "CreatedPointLight",
            },
            Settings = {
                Intensity = 3,
                VolumetricScatteringIntensity = 1,
                ShadowResolutionScale = 0.5,
                AttenuationRadius = 1500.0,
                LightColor = { R = 225, G = 205, B = 175, A = 255 },
                RelativeLocation = { X = 20.0, Y = 147.0, Z = 213.0 },
            }
        },
        -- Japanese Paper Lantern
        BP_BuildObject_Andon_C = {
            Enabled = true,
            IgnoreLightCurve = true, -- search "IgnoreLightCurveExplanation"
            Group = "BuildObjects",
            Templates = {
                "PointLightDefaults",
                "BuildObjectDefaults",
            },
            NameContains = {
                "BP_BuildObject_Andon_C_",
                ".BP_PalTimerPointLightComponent"
            },
            Meshes = {
                {
                    NameContains = {
                        ".SM_Rectangular_Table", -- how this name make sense?
                    },
                    CastShadow = false, -- if you want self-shadows set this to true
                },
            },
            Settings = {
                DefaultIntensity = 15,
                --VolumetricScatteringIntensity = 0.25,
                ShadowResolutionScale = 0.5,
                AttenuationRadius = 700.0,
                LightFalloffExponent = 3.0,
                LightColor = { R = 230, G = 180, B = 145, A = 255 },
                RelativeLocation = { X = 0.0, Y = 0.0, Z = 30.0 }, -- inside middle
                --RelativeLocation = { X = 0.0, Y = 0.0, Z = 50.0 }, -- top
            }
        },
        -- Simple Street Lamp
        BP_BuildObject_Light_LightPole01_C = {
            Enabled = true,
            Group = "BuildObjects",
            Templates = {
                "PointLightDefaults",
                "BuildObjectDefaults",
            },
            NameContains = {
                "BP_BuildObject_Light_LightPole01_C_",
                ".PointLight"
            },
            Meshes = {
                {
                    NameContains = {
                        ".SM_LightPole_A2",
                    },
                    CastShadow = false, -- needed to avoid wierd shadows
                },
            },
            Settings = {
                Intensity = 15.0, -- Vanilla: 30
                --LightColor -- do not override here, can be set in game.
                AttenuationRadius = 1000.0, -- Vanilla: 900
                --RelativeLocation = { X = 0.0, Y = 0.0, Z = 320.0 }, -- Above
                --RelativeLocation = { X = 0.0, Y = 0.0, Z = 185.0 }, -- Under
                RelativeLocation = { X = 0.0, Y = 0.0, Z = 245.0 }, -- Inside
            }
        },
        -- Double Street Lamp
        BP_BuildObject_Light_LightPole02_C = {
            Enabled = true,
            Group = "BuildObjects",
            Templates = {
                "PointLightDefaults",
                "BuildObjectDefaults",
                "CreatedPointLight",
            },
            NameContains = {
                "BP_BuildObject_Light_LightPole02_C_",
                ".PointLight"
            },
            Instances = {
                {
                    Name = "Primary",
                    UseExisting = true,
                    Settings = {
                        --RelativeLocation = { X = -80.0, Y = 0.0, Z = 314.0 }, -- outside
                        RelativeLocation = { X = -47.0, Y = 0.0, Z = 314.0 }, -- middle
                    },
                },
                {
                    Name = "Secondary",
                    Follow = {
                        Source = "Primary",
                        LightColor = true,
                        Visibility = true,
                    },
                    Settings = {
                        --RelativeLocation = { X = 80.0, Y = 0.0, Z = 314.0 }, -- outside
                        RelativeLocation = { X = 47.0, Y = 0.0, Z = 314.0 }, -- middle
                    },
                },
            },
            Settings = {
                Intensity = 10.0, -- Vanilla: 30
                AttenuationRadius = 1000.0, -- Vanilla: 1100
                --LightColor -- do not override here, can be set in game.
                VolumetricScatteringIntensity = 0.25,
            }
        },
        -- Retro Street Lamp
        BP_BuildObject_Light_LightPole03_C = {
            Enabled = true,
            Group = "BuildObjects",
            Templates = {
                "PointLightDefaults",
                "BuildObjectDefaults",
            },
            NameContains = {
                "BP_BuildObject_Light_LightPole03_C_",
                ".PointLight"
            },
            Meshes = {
                {
                    NameContains = {
                        ".SM_LightPole_A5",
                    },
                    CastShadow = false, -- needed to avoid wierd shadows
                },
            },
            Settings = {
                Intensity = 15.0,
                --LightColor -- do not override here, can be set in game.
                AttenuationRadius = 1000.0,
                RelativeLocation = { X = 0.0, Y = 0.0, Z = 330.0 }, -- Inside
            }
        },
        -- Stylish Street Lamp
        BP_BuildObject_Light_LightPole04_C = {
            Enabled = true,
            Group = "BuildObjects",
            Templates = {
                "PointLightDefaults",
                "BuildObjectDefaults",
            },
            NameContains = {
                "BP_BuildObject_Light_LightPole04_C_",
                ".PointLight"
            },
            Settings = {
                Intensity = 15.0,
                --LightColor -- do not override here, can be set in game.
                AttenuationRadius = 1000.0,
                RelativeLocation = { X = 69.0, Y = 0.0, Z = 260.0 }, -- Inside
            }
        },
        -- Fireplace
        BP_BuildObject_Light_FirePlace02_C = {
            Enabled = true,
            Group = "BuildObjects",
            Templates = {
                "PointLightDefaults",
                "BuildObjectDefaults",
            },
            NameContains = {
                "BP_BuildObject_Light_FirePlace02_C_",
                ".PointLight"
            },
            Settings = {
                Intensity = 12.0, -- Vanilla: 20
                VolumetricScatteringIntensity = 1.5,
                RelativeLocation = { X = -2.8, Y = 0.0, Z = 38.3 },
            }
        },
        -- Cooking Pot
        BP_BuildObject_CookingStove_C = {
            Enabled = true,
            Group = "BuildObjects",
            Templates = {
                "PointLightDefaults",
                "BuildObjectDefaults",
                "CreatedPointLight",
                "LightColorFire",
            },
            Instances = {
                {
                    Name = "TopGrill",
                    Settings = {
                        Intensity = 7.5,
                        AttenuationRadius = 300, -- low radius, we don't want double shadows, we just wanna light up the food,
                                                 -- and give some light to the face of the character if character is up close.
                        RelativeLocation = { X = 0.0, Y = 0.0, Z = 110.0 },
                    },
                },
                {
                    Name = "BottomFire",
                    Settings = {
                        RelativeLocation = { X = 0.0, Y = 0.0, Z = 42.0 },
                    },
                },
            },
            Settings = {
                Intensity = 15.0,
                VolumetricScatteringIntensity = 0.5,
                ShadowResolutionScale = 0.1,
                LightFalloffExponent = 4.0,
                AttenuationRadius = 700,
            }
        },
        -- Electric Kitchen
        BP_BuildObject_ElectricKitchen_C = {
            Enabled = true,
            Group = "BuildObjects",
            Templates = {
                "PointLightDefaults",
                "BuildObjectDefaults",
                "CreatedPointLight",
                "LightColorFire",
            },
            Instances = {
                {
                    Name = "TopLamp",
                    Settings = {
                        Intensity = 15.0,
                        AttenuationRadius = 500,
                        SourceRadius = 5,
                        SoftSourceRadius = 55,
                        LightColor = { R = 245, G = 245, B = 215, A = 255 }, -- yellowish
                        --LightColor = { R = 255, G = 175, B = 105, A = 255 }, -- orange/yellowish
                        RelativeLocation = { X = -20.0, Y = -31.0, Z = 162.75 },
                    },
                },
                {
                    Name = "TopGrill",
                    Settings = {
                        Intensity = 5.0,
                        SourceRadius = 50,
                        AttenuationRadius = 100,
                        RelativeLocation = { X = 15.0, Y = -30.0, Z = 135.0 },
                    },
                },
                {
                    Name = "BottomFire",
                    Settings = {
                        RelativeLocation = { X = 15.0, Y = -30.0, Z = 52.0 },
                    },
                },
            },
            Settings = {
                Intensity = 10.0,
                VolumetricScatteringIntensity = 0.5,
                --ShadowResolutionScale = 0.1,
                LightFalloffExponent = 4.0,
                AttenuationRadius = 500,
            }
        },
        -- Heater
        BP_BuildObject_HeaterMedieval_C = {
            Enabled = true,
            Group = "BuildObjects",
            Templates = {
                "PointLightDefaults",
                "BuildObjectDefaults",
                "CreatedPointLight",
                "LightColorFire",
            },
            Instances = {
                {
                    -- Low source radius high intensity = hot
                    Name = "TopHeat",
                    Settings = {
                        Intensity = 100.0,
                        SourceRadius = 5,
                        AttenuationRadius = 250,
                        RelativeLocation = { X = 0.0, Y = 0.0, Z = 110.0 },
                    },
                },
                {
                    Name = "MiddleAmbient",
                    Settings = {
                        IndirectLightingIntensity = 50,
                        ShadowResolutionScale = 0.05,
                        AttenuationRadius = 500,
                        RelativeLocation = { X = 0.0, Y = 0.0, Z = 150.0 },
                    },
                },
                {
                    Name = "BottomHole",
                    Settings = {
                        RelativeLocation = { X = 20.0, Y = 0.0, Z = 20.0 },
                    },
                },
            },
            Settings = {
                Intensity = 5.0,
                VolumetricScatteringIntensity = 0.5,
                LightFalloffExponent = 4.0,
                AttenuationRadius = 700,
            }
        },
        -- Primitive Furnace
        BP_BuildObject_BlastFurnace_C = {
            Enabled = true,
            Group = "BuildObjects",
            Templates = {
                "PointLightDefaults",
                "BuildObjectDefaults",
                "CreatedPointLight",
                "LightColorFire",
            },
            Instances = {
                {
                    -- Low source radius high intensity = hot
                    Name = "TopGrillHeat",
                    Settings = {
                        Intensity = 50.0,
                        SourceRadius = 5,
                        AttenuationRadius = 300,
                        RelativeLocation = { X = 10.0, Y = 0.0, Z = 80.0 },
                    },
                },
                {
                    Name = "TopGrill",
                    Settings = {
                        RelativeLocation = { X = 10.0, Y = 0.0, Z = 80.0 },
                    },
                },
                {
                    Name = "BottomHole",
                    Settings = {
                        Intensity = 15.0,
                        RelativeLocation = { X = 0.0, Y = 0.0, Z = 30.0 },
                    },
                },
            },
            Settings = {
                Intensity = 5.0,
                IndirectLightingIntensity = 25.0,
                VolumetricScatteringIntensity = 0.5,
                ShadowResolutionScale = 0.1,
                LightFalloffExponent = 4.0,
                AttenuationRadius = 700,
            }
        },
        -- Improved Furnace
        BP_BuildObject_BlastFurnace_2_C = {
            Enabled = true,
            Group = "BuildObjects",
            Templates = {
                "PointLightDefaults",
                "BuildObjectDefaults",
                "CreatedPointLight",
                "LightColorFire",
            },
            Instances = {
                {
                    -- Low source radius high intensity = hot
                    Name = "TopMiddleHeat",
                    Settings = {
                        Intensity = 200.0,
                        SourceRadius = 5,
                        AttenuationRadius = 200,
                        RelativeLocation = { X = 0.0, Y = 0.0, Z = 115.0 },
                    },
                },
                {
                    Name = "TopMiddle",
                    Settings = {
                        RelativeLocation = { X = 0.0, Y = 0.0, Z = 115.0 },
                    },
                },
                {
                    Name = "BottomMiddle",
                    Settings = {
                        RelativeLocation = { X = 0.0, Y = -20.0, Z = 55.0 },
                    },
                },
            },
            Settings = {
                Intensity = 35.0,
                VolumetricScatteringIntensity = 0.5,
                LightFalloffExponent = 4.0,
                AttenuationRadius = 700,
            }
        },
        -- Electric Furnace
        BP_BuildObject_BlastFurnace3_C = {
            Enabled = true,
            Group = "BuildObjects",
            Templates = {
                "PointLightDefaults",
                "BuildObjectDefaults",
                "CreatedPointLight",
                "LightColorFire",
            },
            Instances = {
                {
                    -- Low source radius high intensity = hot
                    Name = "TopMiddle",
                    Settings = {
                        Intensity = 200.0,
                        SourceRadius = 5,
                        AttenuationRadius = 200,
                        RelativeLocation = { X = 0.0, Y = 0.0, Z = 105.0 },
                    },
                },
                {
                    Name = "AmbientMiddle",
                    Settings = {
                        Intensity = 5.0,
                        VolumetricScatteringIntensity = 0.1,
                        IndirectLightingIntensity = 50,
                        SourceRadius = 50,
                        AttenuationRadius = 500,
                        RelativeLocation = { X = 60.0, Y = 0.0, Z = 130.0 },
                    },
                },
            },
            Settings = {
                VolumetricScatteringIntensity = 0.5,
                LightFalloffExponent = 4.0,
                AttenuationRadius = 200,
            }
        },
        -- Gigantic Furnace
        BP_BuildObject_BlastFurnace4_C = {
            Enabled = true,
            Group = "BuildObjects",
            Templates = {
                "PointLightDefaults",
                "BuildObjectDefaults",
                "CreatedPointLight",
                "LightColorFire",
            },
            Instances = {
                {
                    -- Low source radius high intensity = hot
                    Name = "TopHeat",
                    Settings = {
                        Intensity = 5000.0,
                        SourceRadius = 5,
                        AttenuationRadius = 300,
                        RelativeLocation = { X = 0.0, Y = 0.0, Z = 365.0 },
                    },
                },
                {
                    Name = "MiddleAmbient",
                    Settings = {
                        Intensity = 10.0,
                        VolumetricScatteringIntensity = 0.1,
                        IndirectLightingIntensity = 50,
                        SourceRadius = 50,
                        SoftSourceRadius = 100,
                        AttenuationRadius = 500,
                        RelativeLocation = { X = 60.0, Y = 0.0, Z = 130.0 },
                    },
                },
                {
                    -- Low source radius high intensity = hot
                    Name = "BottomHeat",
                    Settings = {
                        Intensity = 50000.0,
                        SourceRadius = 5,
                        LightFalloffExponent = 6.0,
                        AttenuationRadius = 140,
                        RelativeLocation = { X = 0.0, Y = 0.0, Z = 50.0 },
                    },
                },
            },
            Settings = {
                VolumetricScatteringIntensity = 0.15,
                LightFalloffExponent = 4.0,
                AttenuationRadius = 200,
            }
        },
        -- Pal Pod
        BP_BuildObject_MedicalPalBed_05_C = {
            Enabled = true,
            Group = "BuildObjects",
            Templates = {
                "PointLightDefaults",
                "BuildObjectDefaults",
                "CreatedPointLight",
                "LightColorCyan",
            },
            Instances = {
                {
                    Name = "TopMiddle",
                    Settings = {
                        RelativeLocation = { X = 0.0, Y = 0.0, Z = 250.0 },
                    },
                },
                {
                    Name = "BottomMiddle",
                    Settings = {
                        RelativeLocation = { X = 0.0, Y = 0.0, Z = 65.0 },
                    },
                },
            },
            Settings = {
                Intensity = 100.0,
                VolumetricScatteringIntensity = 1,
                --ShadowResolutionScale = 0.5,
                LightFalloffExponent = 10.0,
                AttenuationRadius = 500,
            }
        },
        -- Pal Surgery Table
        BP_BuildObject_OperatingTable_C = {
            Enabled = true,
            Group = "BuildObjects",
            Templates = {
                "PointLightDefaults",
                "BuildObjectDefaults",
                "CreatedPointLight",
            },
            Instances = {
                {
                    Name = "TopLeft",
                    Settings = {
                        RelativeLocation = { X = -12.0, Y = 120.0, Z = 169.0 },
                    },
                },
                {
                    Name = "TopRight",
                    Settings = {
                        RelativeLocation = { X = -48.0, Y = 57.0, Z = 169.0 },
                    },
                },
                --{
                --    Name = "MiddleIntense",
                --    Settings = {
                --        Intensity = 100,
                --        VolumetricScatteringIntensity = 0.0,
                --        AttenuationRadius = 200,
                --        RelativeLocation = { X = 50.0, Y = 70.0, Z = 140.0 },
                --    },
                --},
            },
            Settings = {
                Intensity = 15,
                IndirectLightingIntensity = 20,
                VolumetricScatteringIntensity = 0.5,
                SourceRadius = 50,
                SoftSourceRadius = 100,
                LightFalloffExponent = 4.0,
                AttenuationRadius = 800,
                LightColor = { R = 230, G = 230, B = 255, A = 255 },
            }
        },
        -- Emergency Exit Wall Sign
        BP_BuildObject_SignExit_Wall_Iron_C = {
            Enabled = true,
            Group = "BuildObjects",
            Templates = {
                "PointLightDefaults",
                "BuildObjectDefaults",
                "CreatedPointLight",
            },
            Instances = {
                {
                    Name = "PrimaryIntense",
                    Settings = {
                        Intensity = 55,
                        VolumetricScatteringIntensity = 1,
                        SourceRadius = 5,
                        AttenuationRadius = 100,
                        RelativeLocation = { X = 30.0, Y = 0.0, Z = 0.0 },
                    },
                },
                {
                    Name = "SecondaryAmbient",
                    Settings = {
                        Intensity = 3,
                        VolumetricScatteringIntensity = 0,
                        SourceRadius = 50,
                        SoftSourceRadius = 100,
                        AttenuationRadius = 500,
                        RelativeLocation = { X = 70.0, Y = 0.0, Z = 0.0 },
                    },
                },
            },
            Settings = {
                IndirectLightingIntensity = 20.0,
                LightColor = { R = 35, G = 205, B = 115, A = 255 },
            }
        },
        -- Emergency Exit Ceiling Sign
        BP_BuildObject_SignExit_Ceiling_Iron_C = {
            Enabled = true,
            Group = "BuildObjects",
            Templates = {
                "PointLightDefaults",
                "BuildObjectDefaults",
                "CreatedPointLight",
            },
            Instances = {
                {
                    Name = "PrimaryIntense",
                    Settings = {
                        Intensity = 75,
                        VolumetricScatteringIntensity = 1,
                        SourceRadius = 5,
                        AttenuationRadius = 75,
                        RelativeLocation = { X = 25.0, Y = 0.0, Z = -40.0 },
                    },
                },
                {
                    Name = "SecondaryAmbient",
                    Settings = {
                        Intensity = 5,
                        VolumetricScatteringIntensity = 0,
                        SourceRadius = 50,
                        SoftSourceRadius = 100,
                        AttenuationRadius = 300,
                        RelativeLocation = { X = 65.0, Y = 0.0, Z = -40.0 },
                    },
                },
            },
            Settings = {
                IndirectLightingIntensity = 20.0,
                LightColor = { R = 35, G = 205, B = 115, A = 255 },
            }
        },
        -- Clinic
        BP_BuildObject_Clinic_C = {
            Enabled = true,
            Group = "BuildObjects",
            Templates = {
                "PointLightDefaults",
                "BuildObjectDefaults",
                "CreatedPointLight",
            },
            Instances = {
                {
                    Name = "PrimaryIntense",
                    Settings = {
                        Intensity = 10,
                        VolumetricScatteringIntensity = 25,
                        AttenuationRadius = 75,
                        RelativeLocation = { X = 92.0, Y = 120.0, Z = 125.0 },
                    },
                },
                {
                    Name = "SecondaryAmbient",
                    Settings = {
                        Intensity = 5,
                        AttenuationRadius = 800,
                        RelativeLocation = { X = 92.0, Y = 110.0, Z = 125.0 },
                    },
                },
            },
            Settings = {
                LightColor = { R = 255, G = 135, B = 10, A = 255 },
                --LightColor = { R = 205, G = 105, B = 10, A = 255 }, -- orange
                --LightColor = { R = 255, G = 135, B = 10, A = 255 }, -- orange
            }
        },
        -- Antique Washstand
        BP_BuildObject_TableSink01_Stone_C = {
            Enabled = true,
            Group = "BuildObjects",
            Templates = {
                "PointLightDefaults",
                "BuildObjectDefaults",
                "CreatedPointLight",
            },
            Instances = {
                {
                    Name = "TopLeft",
                    Settings = {
                        RelativeLocation = { X = -20.0, Y = 62.5, Z = 215.0 },
                    },
                },
                {
                    Name = "TopRight",
                    Settings = {
                        RelativeLocation = { X = -20.0, Y = -63.0, Z = 215.0 },
                    },
                },
                {
                    Name = "MiddleAmbient",
                    Settings = {
                        Intensity = 6,
                        VolumetricScatteringIntensity = 0.5,
                        AttenuationRadius = 500,
                        RelativeLocation = { X = 20.0, Y = 0.0, Z = 215.0 },
                    },
                },
            },
            Settings = {
                Intensity = 50,
                SourceRadius = 5,
                IndirectLightingIntensity = 20,
                VolumetricScatteringIntensity = 1,
                AttenuationRadius = 100,
                LightColor = { R = 230, G = 230, B = 255, A = 255 },
            }
        },
        -- Electric Medicine Workbench
        BP_BuildObject_MedicineFacility_02_C = {
            Enabled = true,
            Group = "BuildObjects",
            Templates = {
                "PointLightDefaults",
                "BuildObjectDefaults",
                "CreatedPointLight",
            },
            Settings = {
                Intensity = 30,
                SourceRadius = 50,
                IndirectLightingIntensity = 20,
                VolumetricScatteringIntensity = 1,
                LightFalloffExponent = 4.0,
                AttenuationRadius = 400,
                LightColor = { R = 185, G = 125, B = 220, A = 255 },
                RelativeLocation = { X = 20.0, Y = 5.0, Z = 125.0 },
            }
        },
        -- Palbox
        BP_BuildObject_PalBoxV2_C = {
            Enabled = true,
            Group = "BuildObjects",
            Templates = {
                "PointLightDefaults",
                "BuildObjectDefaults",
                "CreatedPointLight",
                "LightColorCyan",
            },
            Settings = {
                Intensity = 50,
                SourceRadius = 50,
                SoftSourceRadius = 100,
                IndirectLightingIntensity = 20,
                VolumetricScatteringIntensity = 1,
                LightFalloffExponent = 4.0,
                AttenuationRadius = 400,
                RelativeLocation = { X = -3.0, Y = 0.0, Z = 140.0 },
            }
        },
        -- Palbox Control Device
        BP_BuildObject_BaseCampWorkerExtraStation_C = {
            Enabled = true,
            Group = "BuildObjects",
            Templates = {
                "PointLightDefaults",
                "BuildObjectDefaults",
                "CreatedPointLight",
                "LightColorCyan",
            },
            Settings = {
                Intensity = 30,
                SourceRadius = 50,
                IndirectLightingIntensity = 20,
                VolumetricScatteringIntensity = 1,
                LightFalloffExponent = 4.0,
                AttenuationRadius = 400,
                RelativeLocation = { X = 10.0, Y = 0.0, Z = 170.0 },
            }
        },
        -- Power Generator
        BP_BuildObject_EnergyGenerator_Electric_C = {
            Enabled = true,
            Group = "BuildObjects",
            Templates = {
                "PointLightDefaults",
                "BuildObjectDefaults",
                "CreatedPointLight",
            },
            Settings = {
                Intensity = 12.0,
                AttenuationRadius = 500,
                LightColor = { R = 130, G = 115, B = 65, A = 255 }, -- Electro yellow
                RelativeLocation = { X = 0.0, Y = 0.0, Z = 190.0 },
            }
        },
        -- Accumulator
        BP_BuildObject_EnergyStorage_Electric_C = {
            Enabled = true,
            Group = "BuildObjects",
            Templates = {
                "PointLightDefaults",
                "BuildObjectDefaults",
                "CreatedPointLight",
                "LightColorCyan",
            },
            Instances = {
                {
                    Name = "TopShelf",
                    Settings = {
                        Intensity = 10,
                        RelativeLocation = { X = 30.0, Y = 0.0, Z = 180.0 },
                    },
                },
                {
                    Name = "MiddleShelf",
                    Settings = {
                        RelativeLocation = { X = 30.0, Y = 0.0, Z = 135.0 },
                    },
                },
                {
                    Name = "BottomShelf",
                    Settings = {
                        RelativeLocation = { X = 30.0, Y = 0.0, Z = 65.0 },
                    },
                },
            },
            Settings = {
                Intensity = 40,
                SourceRadius = 50,
                IndirectLightingIntensity = 20,
                VolumetricScatteringIntensity = 1,
                LightFalloffExponent = 4.0,
                AttenuationRadius = 300,
            }
        },
        -- Large Power Generator
        BP_BuildObject_EnergyGenerator_Large_C = {
            Enabled = true,
            Group = "BuildObjects",
            Templates = {
                "PointLightDefaults",
                "BuildObjectDefaults",
                "CreatedPointLight",
                "LightColorCyan",
            },
            Instances = {
                {
                    Name = "TopAntenna",
                    Settings = {
                        Intensity = 300.0,
                        AttenuationRadius = 300,
                        RelativeLocation = { X = 0.0, Y = 0.0, Z = 450.0 },
                    },
                },
                {
                    Name = "Bottom",
                    Settings = {
                        Intensity = 150.0,
                        AttenuationRadius = 300,
                        VolumetricScatteringIntensity = 0.1,
                        LightColor = { R = 130, G = 115, B = 65, A = 255 }, -- Electro yellow
                        RelativeLocation = { X = 0.0, Y = 0.0, Z = 20.0 },
                    },
                },
            },
            Settings = {
            }
        },
        -- Electric Mine
        BP_BuildObject_Trap_MineElecShock_C = {
            Enabled = true,
            Group = "BuildObjects",
            Templates = {
                "PointLightDefaults",
                "BuildObjectDefaults",
                "CreatedPointLight",
            },
            Settings = {
                Intensity = 6.0,
                SourceRadius = 50,
                VolumetricScatteringIntensity = 2,
                AttenuationRadius = 500,
                LightColor = { R = 130, G = 115, B = 65, A = 255 }, -- Electro yellow
                RelativeLocation = { X = 0.0, Y = 0.0, Z = 30.0 },
            }
        },
        -- Summoning Altar
        BP_BuildObject_Altar_C = {
            Enabled = true,
            Group = "BuildObjects",
            Templates = {
                "PointLightDefaults",
                "BuildObjectDefaults",
                "CreatedPointLight",
            },
            Settings = {
                Intensity = 6.0,
                AttenuationRadius = 1500,
                LightColor = { R = 255, G = 20, B = 20, A = 255 },
                RelativeLocation = { X = 0.0, Y = 0.0, Z = 190.0 },
            }
        },
        -- Dimensional Pal Storage
        BP_BuildObject_DimensionPalStorage_C = {
            Enabled = true,
            Group = "BuildObjects",
            Templates = {
                "PointLightDefaults",
                "BuildObjectDefaults",
                "CreatedPointLight",
                "LightColorCyan",
            },
            Instances = {
                {
                    Name = "FrontCyan",
                    Settings = {
                        Intensity = 20.0,
                        LightFalloffExponent = 4.0,
                        RelativeLocation = { X = 90.0, Y = 0.0, Z = 100.0 },
                    },
                },
                {
                    Name = "MiddleRed",
                    -- The chest is red while private and cyan once shared with
                    -- the guild, so this light follows that state. Asking the
                    -- storage model rather than the chest, because that is
                    -- where the game keeps the flag. Only this instance
                    -- changes, FrontCyan above stays as it is.
                    StateColor = {
                        Model = "DimensionPalStorageModel",
                        Function = "IsPrivateLock",
                        [true] = { R = 255, G = 20, B = 20, A = 255 },  -- private
                        [false] = { R = 0, G = 180, B = 255, A = 255 }, -- shared
                    },
                    Settings = {
                        Intensity = 350.0,
                        LightFalloffExponent = 15.0,
                        LightColor = { R = 255, G = 20, B = 20, A = 255 },
                        RelativeLocation = { X = 0.0, Y = 0.0, Z = 175.0 },
                    },
                },
            },
            Settings = {
                VolumetricScatteringIntensity = 1,
                AttenuationRadius = 400,
            }
        },
        -- Advanced Sphere Assembly Line
        BP_BuildObject_SphereFactory_Black_04_C = {
            Enabled = true,
            Group = "BuildObjects",
            Templates = {
                "PointLightDefaults",
                "BuildObjectDefaults",
                "CreatedPointLight",
                "LightColorCyanPale",
            },
            Instances = {
                {
                    Name = "Computer",
                    Settings = {
                        LightColor = { R = 170, G = 230, B = 200, A = 255 }, -- Light Emerald
                        RelativeLocation = { X = 128.0, Y = -590.0, Z = 170.0 },
                    },
                },
                {
                    Name = "ConveyerArmLeft",
                    Settings = {
                        RelativeLocation = { X = 31.0, Y = 270.0, Z = 150.0 },
                    },
                },
                {
                    Name = "ConveyerArmMiddle",
                    Settings = {
                        RelativeLocation = { X = 44.0, Y = 20.0, Z = 150.0 },
                    },
                },
                {
                    Name = "ConveyerArmRight",
                    Settings = {
                        RelativeLocation = { X = 35.0, Y = -230.0, Z = 150.0 },
                    },
                },
            },
            Settings = {
                Intensity = 7.0,
                VolumetricScatteringIntensity = 0.5,
                LightFalloffExponent = 4.0,
                AttenuationRadius = 300,
            }
        },
        -- High-Pressure Crude Oil Extractor
        BP_BuildObject_OilPump02_C = {
            Enabled = true,
            Group = "BuildObjects",
            Templates = {
                "PointLightDefaults",
                "BuildObjectDefaults",
                "CreatedPointLight",
                "LightColorCyan",
            },
            Instances = {
                {
                    Name = "Front",
                    Settings = {
                        RelativeLocation = { X = 135.0, Y = 0.0, Z = 35.0 },
                    },
                },
                {
                    Name = "Right",
                    Settings = {
                        RelativeLocation = { X = 0.0, Y = -135.0, Z = 35.0 },
                    },
                },
                {
                    Name = "Left",
                    Settings = {
                        RelativeLocation = { X = 0.0, Y = 135.0, Z = 35.0 },
                    },
                },
                
            },
            Settings = {
                Intensity = 7.0,
                VolumetricScatteringIntensity = 0.5,
                LightFalloffExponent = 4.0,
                AttenuationRadius = 800,
            }
        },
        -- Medieval Medicine Workbench
        BP_BuildObject_MedicineFacility_01_C = {
            Enabled = true,
            Group = "BuildObjects",
            Templates = {
                "PointLightDefaults",
                "BuildObjectDefaults",
                "CreatedPointLight",
            },
            Settings = {
                Intensity = 6.0,
                VolumetricScatteringIntensity = 0.5,
                LightFalloffExponent = 4.0,
                AttenuationRadius = 200,
                LightColor = { R = 225, G = 225, B = 225, A = 255 },
                RelativeLocation = { X = 40.0, Y = 0.0, Z = 150.0 },
            }
        },
        -- Guild Chest
        BP_BuildObject_GuildChest_C = {
            Enabled = true,
            Group = "BuildObjects",
            Templates = {
                "PointLightDefaults",
                "BuildObjectDefaults",
                "CreatedPointLight",
                "LightColorCyanPale",
            },
            Settings = {
                Intensity = 14.0,
                LightFalloffExponent = 4.0,
                AttenuationRadius = 300,
                RelativeLocation = { X = 30.0, Y = 0.0, Z = 145.0 }, -- screen
            }
        },
        -- Antique Long Cabinet
        BP_BuildObject_Shelf06_Stone_C = {
            Enabled = true,
            Group = "BuildObjects",
            Templates = {
                "PointLightDefaults",
                "BuildObjectDefaults",
                "CreatedPointLight",
            },
            Instances = {
                {
                    Name = "LampIntense",
                    Settings = {
                        Intensity = 60.0,
                        AttenuationRadius = 80,
                        RelativeLocation = { X = 0.0, Y = 86.2, Z = 165.1 },
                    },
                },
                {
                    Name = "Lamp",
                    Settings = {
                        Intensity = 10.0,
                        RelativeLocation = { X = 0.0, Y = 86.2, Z = 149.1 },
                    },
                },
            },
            Settings = {
                LightColor = { R = 255, G = 200, B = 170, A = 255 },
                IndirectLightingIntensity = 100,
                LightFalloffExponent = 4.0,
                AttenuationRadius = 500,
            }
        },
        -- Antique Dresser - Edit Character Appearance, Edit Character Outfit
        BP_BuildObject_TableDresser01_Stone_C = {
            Enabled = true,
            Group = "BuildObjects",
            Templates = {
                "PointLightDefaults",
                "BuildObjectDefaults",
                "CreatedPointLight",
                "LightColorFirePale",
            },
            Instances = {
                {
                    Name = "LeftCandleIntense",
                    Settings = {
                        Intensity = 100.0,
                        AttenuationRadius = 35,
                        RelativeLocation = { X = -12.0, Y = 57.9, Z = 123.7 },
                    },
                },
                {
                    Name = "LeftCandle",
                    Settings = {
                        RelativeLocation = { X = -12.0, Y = 57.9, Z = 123.7 },
                    },
                },
                {
                    Name = "RightCandleIntense",
                    Settings = {
                        Intensity = 100.0,
                        AttenuationRadius = 35,
                        RelativeLocation = { X = -34.5, Y = -54.8, Z = 138.7 },
                    },
                },
                {
                    Name = "RightCandle",
                    Settings = {
                        RelativeLocation = { X = -34.5, Y = -54.8, Z = 138.7 },
                    },
                },
            },
            Settings = {
                Intensity = 4.0,
                IndirectLightingIntensity = 100,
                LightFalloffExponent = 4.0,
                AttenuationRadius = 500,
            }
        },
        -- Japanese Hearth
        BP_BuildObject_Irori_C = {
            Enabled = true,
            Group = "BuildObjects",
            Templates = {
                "PointLightDefaults",
                "BuildObjectDefaults",
                "CreatedPointLight",
                "LightColorFire",
            },
            Settings = {
                Intensity = 3.0,
                IndirectLightingIntensity = 50,
                LightFalloffExponent = 4.0,
                AttenuationRadius = 386,
                RelativeLocation = { X = 0.1, Y = 0.2, Z = 50.0 },
            }
        },
        -- Stone Lantern
        BP_BuildObject_Toro_C = {
            Enabled = true,
            Group = "BuildObjects",
            Templates = {
                "PointLightDefaults",
                "BuildObjectDefaults",
                "CreatedPointLight",
                "LightColorFirePale",
            },
            Instances = {
                {
                    Name = "AmbientTop",
                    Settings = {
                        Intensity = 4.0,
                        RelativeLocation = { X = 0.1, Y = 0.2, Z = 113.0 },
                    },
                },
                {
                    Name = "IntenseInside",
                    Settings = {
                        Intensity = 200.0,
                        CastShadows = false,
                        CastVolumetricShadow = false,
                        AttenuationRadius = 20,
                        RelativeLocation = { X = 0.1, Y = 0.2, Z = 61.3 },
                    },
                },
                {
                    Name = "AmbientInside",
                    Settings = {
                        RelativeLocation = { X = 0.1, Y = 0.2, Z = 60.0 },
                    },
                },
            },
            Settings = {
                Intensity = 8.0,
                LightFalloffExponent = 4.0,
                AttenuationRadius = 686,
            }
        },
        -- Clean Wall and Door
        BP_BuildObject_SF_DoorWall_02_C = {
            Enabled = true,
            Group = "BuildObjects",
            Templates = {
                "PointLightDefaults",
                "BuildObjectDefaults",
                "CreatedPointLight",
                "LightColorCyan",
            },
            Instances = {
                {
                    Name = "1",
                    Settings = {
                        RelativeLocation = { X = 35.23, Y = 0.0, Z = 250.0 },
                    },
                },
                {
                    Name = "2",
                    Settings = {
                        RelativeLocation = { X = -35.23, Y = 0.0, Z = 250.0 },
                    },
                },
            },
            Settings = {
                Intensity = 10,
                SourceRadius = 5,
                AttenuationRadius = 200,
            }
        },
        
        -- Clean Stairs
        --[[
        BP_BuildObject_SF_Stair_C = {
            Enabled = true,
            Group = "BuildObjects",
            Templates = {
                "PointLightDefaults",
                "BuildObjectDefaults",
                "CreatedPointLight",
                "LightColorCyan",
            },
            Instances = {
                {
                    Name = "7",
                    Settings = {
                        RelativeLocation = { X = -80.0, Y = 0.0, Z = 202.0 },
                    },
                },
                {
                    Name = "5",
                    Settings = {
                        RelativeLocation = { X = -21.0, Y = 0.0, Z = 145.0 },
                    },
                },
                {
                    Name = "3",
                    Settings = {
                        RelativeLocation = { X = 47.0, Y = 0.0, Z = 92.0 },
                    },
                },
                {
                    Name = "1",
                    Settings = {
                        RelativeLocation = { X = 116.0, Y = 0.0, Z = 32.0 },
                    },
                },
            },
            Settings = {
                Intensity = 40,
                SourceRadius = 50,
                IndirectLightingIntensity = 20,
                VolumetricScatteringIntensity = 1,
                LightFalloffExponent = 3.0,
                AttenuationRadius = 200,
            }
        },
        ]]
        
        -- ============================================================
        -- SkillEffects
        -- ============================================================
        -- Spirit Flame
        -- BP_SkillEffect_GhostFlame_C
        BP_SkillEffect_GhostFlame_Bullet_C = {
            Enabled = true,
            Group = "SkillEffects",
            Templates = {
                "PointLightDefaults",
                "SkillEffectDefaults",
                "CreatedPointLight",
            },
            Settings = {
                Intensity = 10,
                LightColor = { R = 195, G = 50, B = 90, A = 255 }, -- redish/purple/pink
            }
        },
        -- Dark Ball
        BP_SkillEffect_DarkBall_C = {
            Enabled = true,
            Group = "SkillEffects",
            Templates = {
                "PointLightDefaults",
                "SkillEffectDefaults",
                "CreatedPointLight",
                "LightColorPurple",
            },
            Settings = {
                Intensity = 25,
            }
        },
        -- Ignis Blast
        BP_SkillEffect_FireBlast_C = {
            Enabled = true,
            Group = "SkillEffects",
            Templates = {
                "PointLightDefaults",
                "SkillEffectDefaults",
                "CreatedPointLight",
                "LightColorFire",
            },
            Settings = {
                Intensity = 25,
            }
        },
        -- Shadow Burst, not ready yet, looks bit odd, need to solve timing issue, maybe decrease pump_ms or separate lower pump_ms for skill effects perhaps?
        --[[
        BP_SkillEffect_DarkWave_C = {
            Enabled = true,
            Group = "SkillEffects",
            Templates = {
                "PointLightDefaults",
                "SkillEffectDefaults",
                "CreatedPointLight",
            },
            --DestroyAfterSeconds = 0.25, -- also need to implement something like this, so lights do not linger
            Settings = {
                Intensity = 40,
                VolumetricScatteringIntensity = 1.5,
                AttenuationRadius = 2000.0,
                LightColor = { R = 130, G = 100, B = 255, A = 255 }, -- purple
            }
        },
        ]]
        -- Spirit Fire
        BP_SkillEffect_FireSeed_C = {
            Enabled = true,
            Group = "SkillEffects",
            Templates = {
                "PointLightDefaults",
                "SkillEffectDefaults",
                "CreatedPointLight",
                "LightColorFire",
            },
            Settings = {
                Intensity = 25,
            }
        },
        -- Spirit Fire detonation bullets
        BP_SkillEffect_FireSeed_Bullet_C = {
            Enabled = true,
            Group = "SkillEffects",
            Templates = {
                "PointLightDefaults",
                "SkillEffectDefaults",
                "CreatedPointLight",
            },
            Settings = {
                Intensity = 7,
                AttenuationRadius = 750.0,
                LightColor = { R = 205, G = 105, B = 10, A = 255 }, -- orange
            }
        },
        -- Jetragon Beam Comet
        BP_SkillEffect_Unique_JumpBeam_Beam_C = {
            Enabled = true,
            Group = "SkillEffects",
            Templates = {
                "PointLightDefaults",
                "SkillEffectDefaults",
                "CreatedPointLight",
                "LightColorPurple",
            },
            Settings = {
                Intensity = 25,
                AttenuationRadius = 1500.0,
            }
        },
        -- Jetragon Aerial Missile
        BP_HomingMissile_C = {
            Enabled = true,
            Group = "SkillEffects",
            Templates = {
                "PointLightDefaults",
                "SkillEffectDefaults",
                "CreatedPointLight",
                "LightColorFire",
            },
            Settings = {
                Intensity = 25,
                AttenuationRadius = 1500.0,
            }
        },
        -- Jetragon Aerial Missile Explosion, not ready yet, looks bit odd, need to solve timing issue, maybe decrease pump_ms or separate lower pump_ms for skill effects perhaps?
        --[[
        BP_Explosion_Missile_C = {
            Enabled = true,
            Group = "SkillEffects",
            Templates = {
                "PointLightDefaults",
                "SkillEffectDefaults",
                "CreatedPointLight",
            },
            --DestroyAfterSeconds = 0.25, -- also need to implement something like this, so lights do not linger
            Settings = {
                Intensity = 50,
                VolumetricScatteringIntensity = 50.0,
                LightColor = { R = 205, G = 105, B = 10, A = 255 }, -- orange
                AttenuationRadius = 2000.0,
            }
        },
        ]]
        -- Meteorain
        BP_SkillEffect_CommetRain_Rock_C = {
            Enabled = true,
            Group = "SkillEffects",
            Templates = {
                "PointLightDefaults",
                "SkillEffectDefaults",
                "CreatedPointLight",
                "LightColorPurple",
            },
            Settings = {
                Intensity = 25,
                AttenuationRadius = 2000.0,
                RelativeLocation = { X = -100.0, Y = 0.0, Z = 0.0 }, -- X coontrols up and down for this one, don't ask me why.
            }
        },
        -- Beam Slicer
        BP_SkillEffect_BeamSlicer_Beam_C = {
            Enabled = true,
            Group = "SkillEffects",
            Templates = {
                "PointLightDefaults",
                "SkillEffectDefaults",
                "CreatedPointLight",
                "LightColorPurple",
            },
            Settings = {
                Intensity = 25,
            }
        },
        -- Beam Slicer Explosion
        BP_SkillEffect_BeamSlicer_Mark_C = {
            Enabled = true,
            Group = "SkillEffects",
            Templates = {
                "PointLightDefaults",
                "SkillEffectDefaults",
                "CreatedPointLight",
                "LightColorPurple",
            },
            Settings = {
                Intensity = 15,
                VolumetricScatteringIntensity = 0.1,
                RelativeLocation = { X = 0.0, Y = 0.0, Z = 100.0 },
            }
        },
        -- Star Mine
        BP_SkillEffect_StarMine_Mine_C = {
            Enabled = true,
            Group = "SkillEffects",
            Templates = {
                "PointLightDefaults",
                "SkillEffectDefaults",
                "CreatedPointLight",
                "LightColorPurple",
            },
            Settings = {
                Intensity = 5,
                VolumetricScatteringIntensity = 0,
            }
        },
        -- Dragon Breath
        BP_SkillEffect_BreathBullet_DragonBreath_C = {
            Enabled = true,
            Group = "SkillEffects",
            Templates = {
                "PointLightDefaults",
                "SkillEffectDefaults",
                "SkillEffect_BreathDefaults",
                "CreatedPointLight",
                "LightColorPurple",
            },
            Settings = {
            },
        },
        -- Ignis Breath
        BP_SkillEffect_BreathBullet_Flamethower_C = {
            Enabled = true,
            Group = "SkillEffects",
            Templates = {
                "PointLightDefaults",
                "SkillEffectDefaults",
                "SkillEffect_BreathDefaults",
                "CreatedPointLight",
                "LightColorFire",
            },
            Settings = {
            },
        },
        -- Fire Ball
        BP_SkillEffectFireBall_C = {
            Enabled = true,
            Group = "SkillEffects",
            Templates = {
                "PointLightDefaults",
                "SkillEffectDefaults",
                "CreatedPointLight",
                "LightColorFire",
            },
            Settings = {
                Intensity = 25,
                AttenuationRadius = 3000,
            }
        },
        -- Flare Storm
        BP_SkillEffect_FlareTornadoBullet_C = {
            Enabled = true,
            Group = "SkillEffects",
            Templates = {
                "PointLightDefaults",
                "SkillEffectDefaults",
                "SkillEffect_TornadoDefaults",
                "CreatedPointLight",
                "LightColorFire",
            },
            Settings = {
            }
        },
        -- TODO: What is skill name? saw wild pal use skill, need to update this
        BP_SkillEffect_Eruption_Bullet_C = {
            Enabled = true,
            Group = "SkillEffects",
            Templates = {
                "PointLightDefaults",
                "SkillEffectDefaults",
                "CreatedPointLight",
                "LightColorFire",
            },
            Settings = {
                Intensity = 25,
                AttenuationRadius = 2000,
                RelativeLocation = { X = 0.0, Y = 0.0, Z = -80.0 },
            }
        },
        
        -- ============================================================
        -- Fast Travel
        -- ============================================================
        -- Fast Travel Tower
        BP_LevelObject_TowerFastTravelPoint_C = {
            Enabled = true,
            Group = "FastTravel",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
            },
            StateColor = {
                Function = "IsUnlocked",
                [true] = { R = 0, G = 180, B = 255, A = 255 },   -- unlocked, cyan
                [false] = { R = 255, G = 135, B = 10, A = 255 }, -- locked, orange
            },
            Settings = {
                Intensity = 15.0,
                AttenuationRadius = 2500.0,
                VolumetricScatteringIntensity = 0.35,
                IndirectLightingIntensity = 1.5,
                RelativeLocation = { X = 0.0, Y = 0.0, Z = 650.0 }, -- top center
                --RelativeLocation = { X = -30.0, Y = 0.0, Z = 90.0 }, -- under middle
                --RelativeLocation = { X = -100.0, Y = 0.0, Z = 170.0 }, -- front snowflake
            }
        },
        -- Watchtower
        BP_LevelObject_UnlockMapPoint_C = {
            -- Alternative: BP_pal_map_small_tower_C, both is on same location
            Enabled = true,
            Group = "FastTravel",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "LightColorCyan",
            },
            Instances = {
                {
                    Name = "TopOfTower",
                    Settings = {
                        RelativeLocation = { X = 175.0, Y = -125.0, Z = 2450.0 },
                        LightColor = { R = 210, G = 115, B = 240, A = 255 }, -- pink/purple
                        AttenuationRadius = 600.0,
                        VolumetricScatteringIntensity = 1.0,
                    },
                },
                {
                    Name = "MiddleCrystal",
                    Settings = {
                        RelativeLocation = { X = 175.0, Y = -125.0, Z = 1050.0 },
                    },
                },
            },
            Settings = {
                MaxDrawDistance = 2000000,
                MaxDistanceFadeRange = 500000,
                Intensity = 10.0,
                AttenuationRadius = 8000.0,
                LightFalloffExponent = 2.0,
                SourceRadius = 50,
                SoftSourceRadius = 100,
                ShadowBias = 0.07,
                ShadowResolutionScale = 1.0,
                VolumetricScatteringIntensity = 0.5,
                IndirectLightingIntensity = 10.0,
            },
        },
        -- Boss Tower
        BP_PalBossTower_C = {
            Enabled = true,
            Group = "FastTravel",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "LightColorCyan",
            },
            Instances = {
                {
                    Name = "MiddleRotatingBall",
                    Settings = {
                        Intensity = 25.0,
                        AttenuationRadius = 4000.0,
                        RelativeLocation = { X = 0.0, Y = 0.0, Z = 2200.0 },
                    },
                },
                {
                    Name = "BottomEntrance",
                    Settings = {
                        RelativeLocation = { X = -470.0, Y = 0.0, Z = 500.0 },
                    },
                },
            },
            Settings = {
                Intensity = 25.0,
                AttenuationRadius = 1000.0,
                SourceRadius = 50.0,
                SoftSourceRadius = 100.0,
                VolumetricScatteringIntensity = 0.25,
                IndirectLightingIntensity = 1.5,
            }
        },
        -- Tower Lock Barrier
        BP_LevelObject_TowerLockBarrier_C = {
            Enabled = true,
            Group = "FastTravel",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
                "LightColorCyan",
            },
            Instances = {
                {
                    Name = "TopRoof",
                    Settings = {
                        Intensity = 20.0,
                        AttenuationRadius = 3000.0,
                        RelativeLocation = { X = 0.0, Y = 0.0, Z = 5000.0 },
                    },
                },
                {
                    Name = "Inside",
                    Settings = {
                        RelativeLocation = { X = 0.0, Y = 0.0, Z = 800.0 },
                    },
                },
            },
            Settings = {
                Intensity = 15.0,
                AttenuationRadius = 1500.0,
                VolumetricScatteringIntensity = 0.1,
                IndirectLightingIntensity = 1.5,
            }
        },
        
        -- ============================================================
        -- World Events
        -- ============================================================
        -- Meteor
        BP_SupplyDropActor_Meteor_C = {
            Enabled = true,
            Group = "WorldEvents",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
            },
            Settings = {
                MaxDrawDistance = 2000000,
                MaxDistanceFadeRange = 500000,
                Intensity = 250.0,
                VolumetricScatteringIntensity = 0.05,
                IndirectLightingIntensity = 1.5,
                AttenuationRadius = 20000.0,
                LightColor = { R = 140, G = 45, B = 215, A = 255 }, -- purple
                RelativeLocation = { X = 0.0, Y = 0.0, Z = -200.0 }, -- under
            }
        },
        -- Mineable Meteor Remains
        BP_MapObject_MeteorDrop_Damagable_C = {
            Enabled = true,
            Group = "WorldEvents",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
            },
            Instances = {
                {
                    Name = "TopHole",
                    Settings = {
                        RelativeLocation = { X = 80.0, Y = -40.0, Z = 190.0 },
                    },
                },
                {
                    Name = "BottomHole",
                    Settings = {
                        RelativeLocation = { X = -90.0, Y = 30.0, Z = 115.0 },
                    },
                },
            },
            Settings = {
                Intensity = 55.0,
                VolumetricScatteringIntensity = 0,
                AttenuationRadius = 2000.0,
                LightColor = { R = 170, G = 90, B = 215, A = 255 }, -- purple
                --RelativeLocation = { X = 0.0, Y = 0.0, Z = 400.0 }, -- above
            }
        },
        -- Supply Drop
        BP_SupplyDropActor_Capsule_C = {
            Enabled = true,
            Group = "WorldEvents",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
            },
            Settings = {
                MaxDrawDistance = 2000000,
                MaxDistanceFadeRange = 500000,
                Intensity = 250.0,
                VolumetricScatteringIntensity = 0.05,
                IndirectLightingIntensity = 1.5,
                AttenuationRadius = 20000.0,
                LightColor = { R = 255, G = 170, B = 100, A = 255 },
                RelativeLocation = { X = 0.0, Y = 0.0, Z = -200.0 }, -- under
            }
        },
        -- Supply Chest
        BP_MapObject_SupplyDrop_C = {
            Enabled = true,
            Group = "WorldEvents",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
            },
            Instances = {
                {
                    Name = "FrontChest",
                    Settings = {
                        RelativeLocation = { X = -60.0, Y = 15.0, Z = 65.0 },
                    },
                },
                {
                    Name = "BackChest",
                    Settings = {
                        RelativeLocation = { X = 90.0, Y = 15.0, Z = 65.0 },
                    },
                },
            },
            Settings = {
                Intensity = 15.0,
                VolumetricScatteringIntensity = 0,
                AttenuationRadius = 1500.0,
                LightColor = { R = 110, G = 205, B = 85, A = 255 }, -- green
            }
        },
        
        -- ============================================================
        -- Dungeons
        -- ============================================================
        -- Dungeon Entrances Flat
        DungeonFixedEntrances = {
            Enabled = true,
            Group = "Dungeons",
            ActorClasses = {
                "BP_DungeonFixedEntrance_grass_1_C",
                "BP_DungeonFixedEntrance_grass_2_C",
                "BP_DungeonFixedEntrance_grass_3_C",
                "BP_DungeonFixedEntrance_grass_4_C",
                "BP_DungeonFixedEntrance_grass_5_C",
                "BP_DungeonFixedEntrance_grass_6_C",
                "BP_DungeonFixedEntrance_grass_7_C",
                "BP_DungeonFixedEntrance_grass_9_C",
                "BP_DungeonFixedEntrance_forest_1_C",
                "BP_DungeonFixedEntrance_forest_2_C",
                "BP_DungeonFixedEntrance_forest_3_C",
                "BP_DungeonFixedEntrance_forest_4_C",
                "BP_DungeonFixedEntrance_forest_5_C",
            },
            Templates = {
                "PointLightDefaults",
                "DungeonPassageDefaults",
                "CreatedPointLight",
                "LightColorCyan",
            },
            Settings = {
                RelativeLocation = { X = 0.0, Y = 0.0, Z = 100.0 },
            },
            -- Optional per-class differences:
            -- Overrides = {
            --     BP_DungeonFixedEntrance_grass_3_C = {
            --         Settings = {
            --             RelativeLocation = { X = 0.0, Y = 0.0, Z = 110.0 },
            --         },
            --     },
            -- },
        },
        -- Dungeon Entrances Cave
        DungeonEntrances = {
            Enabled = true,
            Group = "Dungeons",
            ActorClasses = {
                "BP_DungeonEntrance_Grass_C",
                "BP_DungeonEntrance_Grass_2_C",
                "BP_DungeonEntrance_Forest_C",
            },
            Templates = {
                "PointLightDefaults",
                "DungeonPassageDefaults",
                "CreatedPointLight",
                "LightColorCyan",
            },
            Settings = {
                VolumetricScatteringIntensity = 0,
                RelativeLocation = { X = -650.0, Y = 0.0, Z = 200.0 },
            },
            -- Optional per-class differences:
            -- Overrides = {
            --     BP_DungeonEntrance_Forest_C = {
            --         Settings = {
            --             RelativeLocation = { X = -650.0, Y = 0.0, Z = 210.0 },
            --         },
            --     },
            -- },
        },
        -- Dungeon Exit, both Cave and Flat
        BP_DungeonExit_C = {
            Enabled = true,
            Group = "Dungeons",
            Templates = {
                "PointLightDefaults",
                "DungeonPassageDefaults",
                "CreatedPointLight",
                "LightColorCyan",
            },
            Settings = {
                --RelativeLocation = { X = -750.0, Y = 10.0, Z = 250.0 }, -- portal edge
                RelativeLocation = { X = -1100.0, Y = 10.0, Z = 250.0 }, 
                -- as far in as possible, adjusted for cave dungeon, flat dungeon still looks OK
            }
        },
        BP_MapObject_PickupItem_CaveMushroom_C = {
            Enabled = true,
            Group = "Dungeons",
            Templates = {
                "PointLightDefaults",
                "DungeonClutterDefaults",
            },
            NameContains = {
                "BP_MapObject_PickupItem_CaveMushroom_C_",
                ".PointLight"
            },
            Settings = {
                Intensity = 4,
                RelativeLocation = { X = 30.0, Y = 15.0, Z = 120.0 },
            }
        },
        BP_pal_b00_building_Lordenfel_Candelabra_Hanging_Lit_02_C = {
            Enabled = true,
            Group = "Dungeons",
            Templates = {
                "PointLightDefaults",
                "DungeonClutterDefaults",
            },
            NameContains = {
                "BP_pal_b00_building_Lordenfel_Candelabra_Hanging_Lit_02_C_",
                ".PointLight"
            },
            Settings = {
                Intensity = 3,
                AttenuationRadius = 2000.0,
            }
        },
        BP_pal_b00_building_Lordenfel_Candelabra_Lit_02_C = {
            Enabled = true,
            Group = "Dungeons",
            Templates = {
                "PointLightDefaults",
                "DungeonClutterDefaults",
            },
            NameContains = {
                "BP_pal_b00_building_Lordenfel_Candelabra_Lit_02_C_",
                ".PointLight"
            },
            Settings = {
                Intensity = 4,
                AttenuationRadius = 600.0,
            }
        },
        
        -- ============================================================
        -- Collectibles
        -- ============================================================
        -- Blue Note
        BP_LevelObject_Note_C = {
            Enabled = true,
            Group = "Collectibles",
            Templates = {
                "PointLightDefaults",
                "CollectibleDefaults",
                "CreatedPointLight",
                "LightColorCyan",
            },
            Settings = {
                Intensity = 10.0,
                VolumetricScatteringIntensity = 0.25,
                RelativeLocation = { X = 0.0, Y = 0.0, Z = 120.0 },
            }
        },
        -- Lifmunk Effigy
        BP_LevelObject_Relic_C = {
            Enabled = true,
            Group = "Collectibles",
            Templates = {
                "PointLightDefaults",
                "CollectibleDefaults",
                "CreatedPointLight",
            },
            Settings = {
                Intensity = 5.0,
                VolumetricScatteringIntensity = 0,
                LightColor = { R = 110, G = 205, B = 85, A = 255 }, -- green
                RelativeLocation = { X = 0.0, Y = 0.0, Z = 160.0 },
            }
        },
        -- Lamball Effigy
        BP_LevelObject_Relic_SheepBall_C = {
            Enabled = true,
            Group = "Collectibles",
            Templates = {
                "PointLightDefaults",
                "CollectibleDefaults",
                "CreatedPointLight",
            },
            Settings = {
                Intensity = 5.0,
                VolumetricScatteringIntensity = 0,
                LightColor = { R = 255, G = 205, B = 255, A = 255 }, -- white
                RelativeLocation = { X = 0.0, Y = 0.0, Z = 160.0 },
            }
        },
        -- Cattiva Effigy
        BP_LevelObject_Relic_PinkCat_C = {
            Enabled = true,
            Group = "Collectibles",
            Templates = {
                "PointLightDefaults",
                "CollectibleDefaults",
                "CreatedPointLight",
            },
            Settings = {
                Intensity = 5.0,
                VolumetricScatteringIntensity = 0,
                LightColor = { R = 255, G = 115, B = 235, A = 255 }, -- pink
                RelativeLocation = { X = 0.0, Y = 0.0, Z = 190.0 },
            }
        },
        -- Large Pal Soul
        BP_Item_PalSoul3_C = {
            Enabled = true,
            Group = "Collectibles",
            Templates = {
                "PointLightDefaults",
                "CollectibleDefaults",
                "CreatedPointLight",
            },
            Settings = {
                Intensity = 15.0,
                VolumetricScatteringIntensity = 0,
                LightColor = { R = 200, G = 135, B = 95, A = 255 }, -- yellow/orange
                RelativeLocation = { X = 0.0, Y = 0.0, Z = 160.0 },
            }
        },
        
        -- Disabled treasure chests until I get them to work properly
        --[[
        -- Normal Chest
        BP_TreasureBoxVisual_Grade01_C = {
            Enabled = true,
            Group = "Collectibles",
            Templates = {
                "PointLightDefaults",
                "CollectibleDefaults",
                "CreatedPointLight",
            },
            ComponentOwner = {
                -- Create the point light directly on the nearest cached actor of this
                -- class instead of creating it on this visual/helper actor.
                Class = "BP_MapObject_TreasureBox_C",

                -- Maximum owner distance in Unreal units, normally centimetres.
                -- The nearest cached actor outside this range is ignored.
                MaxDistance = 1000.0,

                -- Logical light slot on each owner actor. Entries using the same owner
                -- actor and slot reuse one component instead of creating duplicates.
                Slot = "TreasureLight",
            },
            Settings = {
                Intensity = 7,
                VolumetricScatteringIntensity = 0.0,
                LightColor = { R = 255, G = 170, B = 80, A = 255 }, -- yellow/orange
                RelativeLocation = { X = 0.0, Y = 0.0, Z = 120.0 }, -- center above
                --RelativeLocation = { X = 0.0, Y = 30.0, Z = 40.0 }, -- front low
            }
        },
        -- Epic Chest
        BP_TreasureBoxVisual_Grade03_C = {
            Enabled = true,
            Group = "Collectibles",
            Templates = {
                "PointLightDefaults",
                "CollectibleDefaults",
                "CreatedPointLight",
            },
            ComponentOwner = {
                -- Create the point light directly on the nearest cached actor of this
                -- class instead of creating it on this visual/helper actor.
                Class = "BP_MapObject_TreasureBox_C",

                -- Maximum owner distance in Unreal units, normally centimetres.
                -- The nearest cached actor outside this range is ignored.
                MaxDistance = 1000.0,

                -- Logical light slot on each owner actor. Entries using the same owner
                -- actor and slot reuse one component instead of creating duplicates.
                Slot = "TreasureLight",
            },
            Settings = {
                Intensity = 7,
                VolumetricScatteringIntensity = 0.0,
                LightColor = { R = 130, G = 100, B = 255, A = 255 }, -- purple
                RelativeLocation = { X = 0.0, Y = 0.0, Z = 120.0 }, -- center above
                --RelativeLocation = { X = 0.0, Y = 30.0, Z = 40.0 }, -- front low
            }
        },
        ]]
        
        -- ============================================================
        -- Settlements
        -- ============================================================
        -- 2x Wall mounted torches found at entrance where player first spawns
        BP_pal_b00_building_Lordenfel_Torch_Wallmounted_Lit_04_C = {
            Enabled = true,
            Group = "Settlements",
            Templates = {
                "PointLightDefaults",
            },
            NameContains = {
                "BP_pal_b00_building_Lordenfel_Torch_Wallmounted_Lit_04_C_",
                ".PointLight"
            },
            Settings = {
                Intensity = 2,
                VolumetricScatteringIntensity = 0.25,
                IndirectLightingIntensity = 50.0,
                AttenuationRadius = 800.0,
                RelativeLocation = { X = 0.0, Y = 30.0, Z = 70.0 },
            }
        },
        -- Braziers on the stairs found where player first spawns
        BP_pal_b00_building_Lordenfel_Brazier_Lit_01_C = {
            Enabled = true,
            Group = "Settlements",
            Templates = {
                "PointLightDefaults",
            },
            NameContains = {
                "BP_pal_b00_building_Lordenfel_Brazier_Lit_01_C_",
                ".PointLight"
            },
            Settings = {
                ShadowResolutionScale = 0.1,
                VolumetricScatteringIntensity = 1.5,
                Intensity = 3,
                IndirectLightingIntensity = 20.0,
                ShadowBias = 0.05,
                AttenuationRadius = 1400.0,
                RelativeLocation = { X = 0.0, Y = 0.0, Z = 55.0 },
            }
        },
        -- Standing torches not braziers... in Small Settlement
        BP_pal_b00_building_Lordenfel_Brazier_Lit_02_C = {
            Enabled = true,
            Group = "Settlements",
            Templates = {
                "PointLightDefaults",
            },
            NameContains = {
                "BP_pal_b00_building_Lordenfel_Brazier_Lit_02_C_",
                ".PointLight"
            },
            Settings = {
                MaxDrawDistance = 5000,
                MaxDistanceFadeRange = 1200,
                CastShadows = false,
                CastVolumetricShadow = false,
                Intensity = 3,
                IndirectLightingIntensity = 0,
                VolumetricScatteringIntensity = 0,
                --ShadowResolutionScale = 0.1,
                --ShadowBias = 0.05,
                AttenuationRadius = 600.0,
                RelativeLocation = { X = 0.0, Y = 0.0, Z = 130.0 },
            }
        },
        -- Wall torches in Small Settlement
        BP_pal_b00_building_Lordenfel_Torch_Wallmounted_Lit_02_C = {
            Enabled = true,
            Group = "Settlements",
            Templates = {
                "PointLightDefaults",
            },
            NameContains = {
                "BP_pal_b00_building_Lordenfel_Torch_Wallmounted_Lit_02_C_",
                ".PointLight"
            },
            Settings = {
                MaxDrawDistance = 5000,
                MaxDistanceFadeRange = 1200,
                CastShadows = false,
                CastVolumetricShadow = false,
                Intensity = 2,
                IndirectLightingIntensity = 0,
                VolumetricScatteringIntensity = 0,
                --ShadowResolutionScale = 0.5,
                AttenuationRadius = 600.0,
            }
        },
        -- Candle stick in Small Settlement
        BP_pal_b00_building_Lordenfel_Candlestick_Lit_01_C = {
            Enabled = true,
            Group = "Settlements",
            Templates = {
                "PointLightDefaults",
            },
            NameContains = {
                "BP_pal_b00_building_Lordenfel_Candlestick_Lit_01_C_",
                ".PointLight"
            },
            Settings = {
                MaxDrawDistance = 1400,
                MaxDistanceFadeRange = 350,
                VolumetricScatteringIntensity = 1,
                Intensity = 3,
                IndirectLightingIntensity = 10.0,
                ShadowResolutionScale = 0.5,
                ShadowBias = 0.5,
                AttenuationRadius = 400.0,
                RelativeLocation = { X = 0.0, Y = -0.0, Z = 35.0 },
            }
        },
        -- Ceiling light in guard tower in Small Settlement
        BP_pal_b00_building_Lordenfel_Candelabra_Wall_Lit_02_C = {
            Enabled = true,
            Group = "Settlements",
            Templates = {
                "PointLightDefaults",
            },
            NameContains = {
                "BP_pal_b00_building_Lordenfel_Candelabra_Wall_Lit_02_C_",
                ".PointLight"
            },
            Settings = {
                Intensity = 5,
                IndirectLightingIntensity = 10.0,
                VolumetricScatteringIntensity = 0.25,
                ShadowResolutionScale = 0.5,
                AttenuationRadius = 800.0,
                RelativeLocation = { X = 0.0, Y = 100.0, Z = -30.0 },
            }
        },
        
        -- ============================================================
        -- World Objects
        -- ============================================================
        BP_LevelGimmickJumpSpotSmall_C = {
            Enabled = true,
            Group = "WorldObjects",
            Templates = {
                "PointLightDefaults",
                "JumpSpotDefaults",
                "CreatedPointLight",
            },
            Settings = {
                AttenuationRadius = 1200.0,
            }
        },
        BP_LevelGimmickJumpSpotLarge_C = {
            Enabled = true,
            Group = "WorldObjects",
            Templates = {
                "PointLightDefaults",
                "JumpSpotDefaults",
                "CreatedPointLight",
            },
            Settings = {
                AttenuationRadius = 1500.0,
            }
        },
        
        -- ============================================================
        -- NPC and Faction Objects
        -- ============================================================
        -- Broken building with lifmunk effigy and two npcs
        -- The Small gas lamp behind green hat lady
        BP_lamp_01_NPC_C = {
            Enabled = true,
            Group = "NPCObjects",
            Templates = {
                "PointLightDefaults",
            },
            NameContains = {
                "BP_lamp_01_NPC_C_",
                ".PointLight"
            },
            Settings = {
                Intensity = 10,
                AttenuationRadius = 600.0,
                RelativeLocation = { X = 0.0, Y = 0.0, Z = 40.0 }, -- above
            }
        },
        -- The small campfire with cooking pot
        BP_Fireplace_NPC_C = {
            Enabled = true,
            IgnoreLightCurve = false, -- search "IgnoreLightCurveExplanation"
            Group = "NPCObjects",
            Templates = {
                "PointLightDefaults",
            },
            NameContains = {
                "BP_Fireplace_NPC_C_",
                ".BP_PalTimerPointLightComponent"
            },
            Settings = {
                DefaultIntensity = 10.0,
                VolumetricScatteringIntensity = 0.25,
                AttenuationRadius = 1400.0,
                RelativeLocation = { X = 0.0, Y = 0.55, Z = 100.0 },
            }
        },
        -- Gang Torch
        BP_GangCamp_GangTorch_C = {
            Enabled = true,
            Group = "NPCObjects",
            Templates = {
                "PointLightDefaults",
            },
            NameContains = {
                "BP_GangCamp_GangTorch_C_",
                ".PointLight"
            },
            Settings = {
                Intensity = 4,
                VolumetricScatteringIntensity = 0.25,
                IndirectLightingIntensity = 10.0,
                AttenuationRadius = 1000.0,
                RelativeLocation = { X = 2.0, Y = -2.0, Z = 130.0 },
            }
        },
        -- Gang Fireplace
        BP_Fireplace_C = {
            Enabled = true,
            Group = "NPCObjects",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
            },
            Settings = {
                Intensity = 8,
                VolumetricScatteringIntensity = 0.35,
                IndirectLightingIntensity = 10.0,
                AttenuationRadius = 800.0,
                LightColor = { R = 255, G = 170, B = 100, A = 255 },
                RelativeLocation = { X = 0.0, Y = 0.0, Z = 39.0 },
            }
        },
        -- Free Pal Alliance Camp Torch
        BP_pal_b00_building_Lordenfel_Tall_Torch_Lit_01_C = {
            Enabled = true,
            Group = "NPCObjects",
            Templates = {
                "PointLightDefaults",
            },
            NameContains = {
                "BP_pal_b00_building_Lordenfel_Tall_Torch_Lit_01_C_",
                ".PointLight"
            },
            Settings = {
                Intensity = 4,
                VolumetricScatteringIntensity = 0.25,
                IndirectLightingIntensity = 10.0,
                AttenuationRadius = 1000.0,
                RelativeLocation = { X = 0.0, Y = 0.0, Z = 190.0 },
            }
        },
        -- Andon standing lantern
        BP_pal_Shrine_Lantern_C = {
            Enabled = true,
            Group = "NPCObjects",
            Templates = {
                "PointLightDefaults",
            },
            NameContains = {
                "BP_pal_Shrine_Lantern_C_",
                ".PointLight"
            },
            Meshes = {
                {
                    NameContains = {
                        ".SM_pal_b00_ShrineLantern01",
                    },
                    CastShadow = false, -- no self-shadow
                },
            },
            Settings = {
                MaxDrawDistance = 5000,
                MaxDistanceFadeRange = 1200,
                Intensity = 10,
                VolumetricScatteringIntensity = 0.25,
                IndirectLightingIntensity = 10.0,
                ShadowResolutionScale = 0.5,
                AttenuationRadius = 1200.0,
                --RelativeLocation = { X = 0.0, Y = 0.0, Z = 290.0 }, -- above
                RelativeLocation = { X = 0.0, Y = 0.0, Z = 175.0 }, -- inside
            }
        },
        -- Small Andon Lamp, found at -630, 225
        BP_pal_Andon_lamp_01_C = {
            Enabled = true,
            Group = "NPCObjects",
            Templates = {
                "PointLightDefaults",
            },
            NameContains = {
                "BP_pal_Andon_lamp_01_C_",
                ".PointLight"
            },
            Settings = {
                Intensity = 4,
                VolumetricScatteringIntensity = 0.25,
                IndirectLightingIntensity = 10.0,
                ShadowResolutionScale = 0.1,
                AttenuationRadius = 800.0,
                RelativeLocation = { X = 0.0, Y = 0.0, Z = 23.0 }, -- inside middle
            }
        },
        -- Medium Andon Lamp, found at -630, 225
        BP_pal_Andon_lamp_02_C = {
            Enabled = true,
            Group = "NPCObjects",
            Templates = {
                "PointLightDefaults",
            },
            NameContains = {
                "BP_pal_Andon_lamp_02_C_",
                ".PointLight"
            },
            Settings = {
                Intensity = 10,
                VolumetricScatteringIntensity = 0.25,
                IndirectLightingIntensity = 10.0,
                ShadowResolutionScale = 0.1,
                AttenuationRadius = 800.0,
                RelativeLocation = { X = 0.0, Y = 0.0, Z = 30.0 }, -- inside middle
            }
        },
        -- These walls will have a andon lamp hanging to the top left
        -- and thats where we will place this light
        -- This building with these walls can be found here: -692, 189
        BP_EnemyCampObject_JapaneseStyle_DoorWall_01_C = {
            Enabled = true,
            Group = "NPCObjects",
            Templates = {
                "PointLightDefaults",
                "CreatedPointLight",
            },
            Settings = {
                Intensity = 5,
                VolumetricScatteringIntensity = 1,
                IndirectLightingIntensity = 10.0,
                ShadowResolutionScale = 0.5,
                AttenuationRadius = 1500.0,
                LightColor = { R = 225, G = 205, B = 175, A = 255 },
                RelativeLocation = { X = 20.0, Y = 147.0, Z = 213.0 },
            }
        },
    },
    
    -- Master switch for the changelog notice on the title screen. This is no
    -- longer rewritten by the mod: the version you have already seen is
    -- recorded in UltraGraphics_State.ini instead, so the notice appears once
    -- per new version and then stops on its own. Set this to false to turn it
    -- off permanently.
    ShowChangelog = true,

UpdateAlertMessage = [[
- Fixed an issue that caused Lumen to flicker during sunrise and sunset.

- Fixed an issue that caused black visual noise when using Lumen.

- Dimensional Pal Storage now lights up red while private and cyan once shared with the guild.

<Yellow_20B>Changelog 1.2.1</>
- Added configurable locked and unlocked light colors for Fast Travel Tower. Thanks to Bartoneye for investigating the tower unlock behavior.

- Added light configuration for Pal Spheres. Thanks to Bartoneye for helping with this work.

- Added light configuration for more build objects.

- Japanese Paper Lantern and glowing pals now stay lit during the day instead of following the game's day and night curve. This can be turned off per light with IgnoreLightCurve in config.lua.

- Firearms now light up their surroundings when fired. Each weapon can be adjusted, or turned off, under the new Weapons light group in config.lua.
<Yellow_20B>Changelog 1.2.0</>
- Removed weather, it now belongs in the mod UltraWeather.

- Enabled lumen reflections.

- Added CVars for higher quality shadows, off by default.

- Added CVars for glass reflections, off by default.

- Adjusted values of several light sources to make them compatible with the new shadows.

<Yellow_20B>Changelog 1.1.9</>
- The default hotkeys have changed. You can customize all three under Keys in config.lua.

- Added <Blue_16>Performance Mode</>, which can be toggled with F10.

- Toggle states are now remembered between sessions and saved in <Blue_16>UltraGraphics_State.ini</>, located next to config.lua.
]],

}

-- ============================================================
-- INTERNAL CONFIGURATION
-- Advanced users may edit below. Normal users can stop here.
-- ============================================================

-- Tables with consecutive numeric keys are configuration arrays such as
-- Templates, NameContains and Instances. Arrays replace earlier arrays rather
-- than merging element-by-element; normal keyed tables are deep-merged.
local function IsArray(value)
    if type(value) ~= "table" then return false end

    local count = 0
    for key in pairs(value) do
        if type(key) ~= "number" or key < 1 or key % 1 ~= 0 then
            return false
        end
        count = count + 1
    end

    if count == 0 then return false end
    for index = 1, count do
        if rawget(value, index) == nil then return false end
    end
    return true
end

local function CopyConfigValue(value)
    if type(value) ~= "table" then return value end

    local result = {}
    for key, child in pairs(value) do
        result[key] = CopyConfigValue(child)
    end
    return result
end

local function MergeConfigValues(target, source)
    for key, value in pairs(source or {}) do
        local current = target[key]
        if type(value) == "table" and type(current) == "table"
                and not IsArray(value) and not IsArray(current) then
            MergeConfigValues(current, value)
        else
            target[key] = CopyConfigValue(value)
        end
    end
    return target
end

-- Expand config-only actor families into normal exact actor-class entries.
-- A family provides shared fields, templates and settings through ActorClasses.
-- Optional Overrides are merged last for individual classes. The family entry
-- is removed afterward so main.lua still receives its expected flat table.
local function ExpandLightActorFamilies(config)
    local lights = config.Lights or {}
    local families = {}

    for familyName, entry in pairs(lights) do
        if type(entry) == "table" and entry.ActorClasses ~= nil then
            families[#families + 1] = {
                Name = familyName,
                Entry = entry,
            }
        end
    end

    for _, family in ipairs(families) do
        local familyName = family.Name
        local familyEntry = family.Entry
        local actorClasses = familyEntry.ActorClasses
        local overrides = familyEntry.Overrides or {}

        assert(IsArray(actorClasses),
            ("ActorClasses on %s must be an ordered array of actor-class names")
                :format(tostring(familyName)))
        assert(type(overrides) == "table" and not IsArray(overrides),
            ("Overrides on %s must be a table keyed by actor class")
                :format(tostring(familyName)))

        local familyBase = CopyConfigValue(familyEntry)
        familyBase.ActorClasses = nil
        familyBase.Overrides = nil

        local listedClasses = {}
        for index, actorClass in ipairs(actorClasses) do
            assert(type(actorClass) == "string" and actorClass ~= "",
                ("ActorClasses entry %d on %s must be a non-empty string")
                    :format(index, tostring(familyName)))
            assert(not listedClasses[actorClass],
                ("Actor class '%s' is listed more than once on %s")
                    :format(actorClass, tostring(familyName)))
            assert(lights[actorClass] == nil,
                ("Actor family %s cannot create '%s': that key already exists")
                    :format(tostring(familyName), actorClass))

            listedClasses[actorClass] = true
            local expandedEntry = CopyConfigValue(familyBase)
            local actorOverride = overrides[actorClass]
            if actorOverride ~= nil then
                assert(type(actorOverride) == "table",
                    ("Override for '%s' on %s must be a table")
                        :format(actorClass, tostring(familyName)))
                MergeConfigValues(expandedEntry, actorOverride)
            end
            lights[actorClass] = expandedEntry
        end

        for actorClass in pairs(overrides) do
            assert(listedClasses[actorClass],
                ("Override for '%s' on %s has no matching ActorClasses entry")
                    :format(tostring(actorClass), tostring(familyName)))
        end

        lights[familyName] = nil
    end
end

-- Resolve each actor's explicit template list from left to right, then apply
-- the actor entry itself last. Templates cannot inherit other templates, so
-- every dependency remains visible on the actor entry. Remove template metadata
-- afterward so main.lua receives the same flat runtime format it used before.
local function ApplyLightTemplates(config)
    local lights = config.Lights or {}
    local templates = lights.Templates or {}
    lights.Templates = nil

    local function ValidateTemplateName(templateName, context)
        assert(type(templateName) == "string",
            ("Template name in %s must be a string"):format(context))
        local template = templates[templateName]
        assert(template ~= nil,
            ("Unknown light template '%s' in %s")
                :format(templateName, context))
        return template
    end

    local function ValidateTemplateList(templateNames, context)
        assert(IsArray(templateNames),
            ("%s must be an ordered array of template names")
                :format(context))
        for index, templateName in ipairs(templateNames) do
            assert(type(templateName) == "string",
                ("Template %d in %s must be a name")
                    :format(index, context))
            ValidateTemplateName(templateName, context)
        end
    end

    for templateName, template in pairs(templates) do
        assert(type(template) == "table",
            ("Light template '%s' must be a table"):format(
                tostring(templateName)))
        assert(template.Template == nil and template.Templates == nil,
            ("Light template '%s' cannot inherit other templates"):format(
                tostring(templateName)))
    end


    for actorClass, entry in pairs(lights) do
        if actorClass ~= "Enabled"
                and actorClass ~= "Groups"
                and actorClass ~= "DisableLightCulling"
                and type(entry) == "table" then
            local explicitNames = entry.Templates
            if explicitNames ~= nil then
                ValidateTemplateList(explicitNames,
                    ("Templates on %s"):format(tostring(actorClass)))
            end

            local resolved = {}

            for _, templateName in ipairs(explicitNames or {}) do
                MergeConfigValues(resolved, templates[templateName])
            end

            local actorOverrides = CopyConfigValue(entry)
            actorOverrides.Templates = nil
            MergeConfigValues(resolved, actorOverrides)
            lights[actorClass] = resolved
        end
    end
end

-- Apply category switches after templates have produced the complete entries.
-- Groups is user-facing metadata, so remove it before returning. Disabled actor
-- entries remain present for created-light cleanup, culling restoration and
-- spawn-notifier registration.
local function ApplyLightGroupSwitches(config)
    local lights = config.Lights or {}
    local groups = lights.Groups or {}
    lights.Groups = nil

    for actorClass, entry in pairs(lights) do
        if actorClass ~= "Enabled"
                and actorClass ~= "DisableLightCulling"
                and type(entry) == "table" then
            local group = entry.Group
            if group ~= nil then
                assert(groups[group] ~= nil,
                    ("Unknown Lights.Groups category '%s' on %s")
                        :format(tostring(group), tostring(actorClass)))
                entry.Enabled = entry.Enabled ~= false and groups[group] ~= false
            end
        end
    end
end

ExpandLightActorFamilies(Config)
ApplyLightTemplates(Config)
ApplyLightGroupSwitches(Config)
return Config
