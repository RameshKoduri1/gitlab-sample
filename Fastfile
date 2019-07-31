# This file contains the fastlane.tools configuration
# You can find the documentation at https://docs.fastlane.tools
#
# For a list of all available actions, check out
#
#     https://docs.fastlane.tools/actions
#
# For a list of all available plugins, check out
#
#     https://docs.fastlane.tools/plugins/available-plugins
#

# Uncomment the line if you want fastlane to automatically update itself
# update_fastlane

desc "Install certificates and provisioining profiles"
lane :certificates do |options|
    if options[:use_temporary_keychain]
        create_temp_keychain
    end     
    # Set read only parameter
    readonly = (options[:refresh_certificates] ? false : true)

    match(
        git_branch: "ci",
        app_identifier: ["com.bayer.bbs.bADPCTEST"],
        # team_id: "BXC5N2D83U",
        type: "development",
        readonly: readonly,
        # clone_branch_directly: true,
        shallow_clone: true,
        verbose: true
    )
    if options[:use_temporary_keychain]
        sh "security set-key-partition-list -S apple-tool:,apple:,codesign: -s -k #{ENV["KEYCHAIN_NAME"]} #{ENV["KEYCHAIN_PASSWORD"]}"
      end
end

# Build and archive application
lane :beta do
  # disable_automatic_code_signing(
  #   path: "PassMan.xcodeproj"
  # )
  # automatic_code_signing(
  #   path: "PassMan.xcodeproj",
  #   use_automatic_signing: false
  # )
  upgrade_super_old_xcode_project(
    path: "PassMan.xcodeproj",
    team_id: "BXC5N2D83U"
  )

  update_app_identifier(
    xcodeproj: "PassMan.xcodeproj", # Optional path to xcodeproj, will use the first .xcodeproj if not set
    plist_path: "PassMan/PassMan-Info.plist", # Path to info plist file, relative to xcodeproj
    app_identifier: "com.bayer.bbs.bADPCTEST" # The App Identifier
  )

  update_project_team(
    path: "PassMan.xcodeproj",
    teamid: "BXC5N2D83U"
  )

  update_project_provisioning(
    xcodeproj: "PassMan.xcodeproj",
    target_filter: "PassMan",
    profile: "./fastlane/2019_BAD_PC_TEST_DEV.mobileprovision",
    build_configuration: "Release",
    code_signing_identity: "iPhone Developer: Ramesh Koduri (Z3KF637SP7)"
  )

  build_app(
    export_method: "development",
    scheme: "PassMan",
    configuration: "Release",
    output_directory: "./output", 
    output_name: "PassMan.ipa",
    codesigning_identity: "iPhone Developer: Ramesh Koduri (Z3KF637SP7)",
    export_options: {
        team_id: "BXC5N2D83U",
        method: "development",
        provisioningProfiles: { 
          "com.bayer.bbs.bADPCTEST" => "2019 BAD PC TEST DEV"
        }
    }
  )
end

lane :distribute do
  hockey(
         api_token: "deec6671c5424b96b8bab1747cc0f6eb",
         ipa: "./output/PassMan.ipa",
         notes: "Changelog"
  )
end

private_lane :create_temp_keychain do
    
  keychain_name = "bayer"
  ENV["KEYCHAIN_NAME"] = keychain_name
  ENV["KEYCHAIN_PASSWORD"] = keychain_name
  ENV["MATCH_KEYCHAIN_NAME"] = keychain_name 
  ENV["MATCH_KEYCHAIN_PASSWORD"] = keychain_name

  create_keychain(
    name: keychain_name,
    default_keychain: true,
    unlock: true,
    timeout: 3600,
    add_to_search_list: true
  )
end