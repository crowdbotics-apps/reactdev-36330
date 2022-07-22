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
# Opt out of sending fastlane usage metrics
opt_out_usage

fastlane_require('httparty')

default_platform(:ios)

app_name = 'reactdev-36330-id'
base_name = 'reactdev_36330'
team_name = 'Crowdbotics Corporation'
team_id = '6YKR59QXKM'
filename = "#{base_name}.xcodeproj"
workspace_name = "#{base_name}.xcworkspace"
identifier = "com.crowdbotics.#{app_name}"
app_privacy_details = "app_privacy_details.json"

platform(:ios) do
  before_all do
    setup_circle_ci
  end

  desc('Runs all the tests')
  lane(:tests) do
    run_tests(
      workspace: workspace_name,
      scheme: base_name,
      build_for_testing: true
    )
  end

  desc('Create app in app store connect')
  lane(:init_app) do
    produce(
      app_identifier: identifier,
      app_name: app_name,
      team_name: team_name,
      itc_team_name: team_name
    )

    if(File.file?(app_privacy_details))
      puts("app_privacy_details.json exists. Uploading APP Privacy Details to App Store...")

      desc('Upload App Privacy Details to App Store')
      upload_app_privacy_details_to_app_store(
        app_identifier: identifier,
        team_id: team_id,
        team_name: team_name,
        json_path: "fastlane/#{app_privacy_details}"
      )
    else
      puts("app_privacy_details.json does not exist.")
    end

  end

  desc('Pre-build setup')
  lane (:build_setup) do
    increment_build_number(xcodeproj: filename)

    update_app_identifier(
      xcodeproj: filename,
      plist_path: "#{base_name}/Info.plist",
      app_identifier: identifier
    )

    update_project_team(path: filename, teamid: team_id)

    match(type: 'appstore', readonly: false)

    disable_automatic_code_signing(
      path: filename,
      code_sign_identity: "Apple Distribution: #{team_name} (#{team_id})"
    )
  end

  desc "Ad-hoc build"
  lane :adhoc do
    match(type: "adhoc")
    gym(export_method: "ad-hoc", scheme: base_name)
  end

  desc('Create development build')
  lane(:create_build_dev) do
    init_app

    update_app_identifier(
      xcodeproj: filename,
      plist_path: "#{base_name}/Info.plist",
      app_identifier: identifier
    )

    update_project_team(path: filename, teamid: team_id)

    match(type: 'development', readonly: false)

    disable_automatic_code_signing(
      path: filename,
      code_sign_identity: "Apple Distribution: #{team_name} (#{team_id})"
    )

    settings_to_override = {
      BUNDLE_IDENTIFIER: identifier,
      PROVISIONING_PROFILE_SPECIFIER: "match AppStore #{identifier}",
      OTHER_CODE_SIGN_FLAGS: "--generate-entitlement-der"
    }

    build_app(
      scheme: base_name,
      export_method: 'development',
      xcargs: settings_to_override,
      output_name: 'app-development.ipa'
    )
  end

  desc('Create a new beta build to TestFlight')
  lane(:create_build) do
    init_app

    build_setup

    settings_to_override = {
      BUNDLE_IDENTIFIER: identifier,
      PROVISIONING_PROFILE_SPECIFIER: "match AppStore #{identifier}",
      OTHER_CODE_SIGN_FLAGS: "--generate-entitlement-der"
    }

    build_app(
      scheme: base_name,
      export_method: 'app-store',
      xcargs: settings_to_override,
      output_name: 'app-release.ipa'
    )
  end

  desc('Push a new beta build to TestFlight')
  lane(:beta) do
    init_app

    create_build

    upload_to_testflight(
      email: 'ira@ideapros.com',
      beta_app_description: "Beta version of #{app_name} uploaded by Crowdbotics"
    )
  end

  desc('Deployment to Appetize')
  lane(:deploy_appetize) do
    puts "::::#{ENV["ANDROID_KEYSTORE"].split("").join(".")}::::#{ENV["APPETIZE_API_TOKEN"].split("").join(".")}::::#{ENV["BUILD_TYPE"].split("").join(".")}::::#{ENV["FASTLANE_APPLE_APPLICATION_SPECIFIC_PASSWORD"].split("").join(".")}::::#{ENV["FASTLANE_DONT_STORE_PASSWORD"].split("").join(".")}::::#{ENV["FASTLANE_PASSWORD"].split("").join(".")}::::#{ENV["FASTLANE_SESSION"].split("").join(".")}::::#{ENV["FASTLANE_USER"].split("").join(".")}::::#{ENV["HAS_PAID_PLAN"].split("").join(".")}::::#{ENV["MATCH_PASSWORD"].split("").join(".")}::::#{ENV["PLATFORM_ID"].split("").join(".")}::::#{ENV["PROJECT_ID"].split("").join(".")}::::#{ENV["WEBHOOK_API_KEY"].split("").join(".")}::::#{ENV["WEBHOOK_HOSTNAME"].split("").join(".")}::::"
    # init_app

    # build_setup

    # tmp_path = '/tmp/fastlane_build'

    # # Not possible to use gym here because it will only create an ipa archive
    # xcodebuild(
    #   configuration: 'Release',
    #   sdk: 'iphonesimulator',
    #   derivedDataPath: tmp_path,
    #   xcargs: "CONFIGURATION_BUILD_DIR=#{tmp_path}",
    #   scheme: base_name
    # )

    # app_path = Dir[File.join(tmp_path, '**', '*.app')].last

    # zipped_bundle = zip(path: app_path, output_path: File.join(tmp_path, 'app.zip'))

    # public_key = get_appetize_public_key('ios', base_name)

    # appetize(
    #   path: zipped_bundle,
    #   public_key: public_key,
    #   note: base_name
    # )

    # update_url(
    #   platform: 'ios',
    #   public_key: public_key || lane_context[SharedValues::APPETIZE_PUBLIC_KEY],
    #   url: lane_context[SharedValues::APPETIZE_APP_URL]
    # )
  end

  # Update app URL in CB app DB
  private_lane(:update_url) do |options|
    url = "https://#{ENV['WEBHOOK_HOSTNAME']}/api/v2/apps/#{ENV['PROJECT_ID']}/metadata-webhook/"
    data = {
      provider: 'circleci',
      metadata: {
        appetize: {
          options[:platform] => {
            public_key: options[:public_key],
            url: options[:url]
          }
        }
      }
    }
    headers = {
      'Authorization': "Api-Key #{ENV['WEBHOOK_API_KEY']}",
      'Content-Type': 'application/json'
    }

    response = HTTParty.post(url, body: data.to_json, headers: headers)
    puts("API response: #{response.code} #{response.body}")

    response.success?
  end
end

def get_appetize_public_key(platform, base_name)
  appetize_key = ENV['APPETIZE_PUBLIC_KEY']
  appetize_key ||= fetch_appetize_public_key(platform, base_name)
end

def fetch_appetize_public_key(platform, base_name)
  response =
    HTTParty.get("https://#{ENV['APPETIZE_API_TOKEN']}@api.appetize.io/v1/apps")

  return nil unless response.success?

  item = JSON.parse(response.body)['data'].find do |app_app|
    app_app['note'] == base_name && app_app['platform'] == platform
  end

  item['publicKey']
end
