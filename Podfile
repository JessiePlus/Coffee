
source 'https://github.com/CocoaPods/Specs.git'
platform :ios, '9.0'
use_frameworks!

def pods
    pod 'KeypathObserver'
    pod 'AutoReview'
    pod 'AudioBot'
    pod 'AsyncDisplayKit'
    pod 'Appsee'
    pod 'DeviceGuru'
    pod 'FXBlurView'
    pod 'TPKeyboardAvoiding'
    pod 'pop'
    pod 'Base64'
    pod 'SocketRocket'
    pod 'JPush'
    pod 'Fabric'
end

target ‘Coffee’ do
    pods

    target 'CoffeeTests' do
        inherit! :search_paths
    end
end

post_install do |installer|
    puts 'Allow app extension api only:'
    installer.pods_project.targets.each do |target|
        case target.name
        when 'Base64', 'SocketRocket'
            target.build_configurations.each do |config|
                config.build_settings['APPLICATION_EXTENSION_API_ONLY'] = 'YES'
                puts 'X...' + target.name
            end
        else
            puts '....' + target.name
        end
    end
end
