fastlane_require 'dotenv'
skip_docs
default_platform(:ios)

before_all do
  Dotenv.overload('../.env.fastlane', '.env')
end

platform :ios do
  def sources
    return ENV['POD_SOURCES'].split(",")    
  end

  desc 'Validate the project is ready for releasing'
  lane :lint do
    sh("cd .. && mint run realm/SwiftLint@0.40.3 swiftlint autocorrect --format || :")

    pod_lib_lint(
      allow_warnings: true,
      sources: sources(),
      private: true
    )
  end

  desc 'Release a new version with a `patch` bump_type'
  lane :patch do
    release('patch')
  end

  desc 'Release a new version with a `minor` bump_type'
  lane :minor do
    release('minor')
  end

  desc 'Release a new version with a `major` bump_type'
  lane :major do
    release('major')
  end

  def release(type)
    new_changes = sh("git log #{last_git_tag}..HEAD | wc -l").strip!
    if new_changes == '0'
      UI.user_error!("No changes since last release: #{last_git_tag}, please add new features and try again!")
    end

    podspec = ENV['PODSPEC']
    podrepo = ENV['POD_REPO']
    sources_paths = ENV['SOURCES_PATHS'].split(',')
    git_path = [podspec] and sources_paths

    version = version_bump_podspec(
      path: podspec,
      bump_type: type
    )

    git_add(
      path: git_path,
      shell_escape: false
    )

    git_commit(
      path: git_path,
      message: "release: v#{version}"
    )

    add_git_tag(
      tag: "#{version}"
    )

    push_to_git_remote

    skip_import_validation = is_ci
    pod_push(
      path: podspec,
      repo: podrepo,
      sources: sources(),
      use_bundle_exec: true,
      allow_warnings: true,
      skip_import_validation: skip_import_validation
    )

    if is_ci
      author = last_git_commit[:author]
      repo_url = ENV['REPO_URL']
      alert("*#{ENV['NAME']} v#{version} is here 🎉*", { :'Download URL' => repo_url, :Author => author })
    end
  end

  desc "Runs all the unit and ui tests"
  lane :test do
    workspace = ENV['WORKSPACE']
    scheme = ENV['SCHEME']
    no_ci = !is_ci

    cocoapods(
      podfile: "./Example/Podfile",
      repo_update: false
    )
    
    scan(
      workspace: workspace,
      scheme: scheme,
      skip_slack: no_ci,
    )
    
    if no_ci
      check_xcov
    end
  end

  def check_xcov()
    workspace = ENV['WORKSPACE']
    scheme = ENV['SCHEME']
    skip_slack = !is_ci

    # Clear folder to solve error
    sh("rm -rf ./xcov_report")

    # Code coverage report
    xcov(
      workspace: workspace,
      scheme: scheme,
      minimum_coverage_percentage: ENV['MIN_COV'].to_f,
      skip_slack: skip_slack
    )
  end

  def alert(message, payload)
    payload['Date'] = Time.new.to_s
    if ENV['SLACK_URL'] 
      slack(
        message: message,
        payload: payload,
        default_payloads: []
      )
    end

    teams_url = ENV['TEAMS_URL'] 
    if teams_url
      facts = payload.collect { |k, v| { :name => k, :value => v  } }
      teams(
        title: message,
        message: "",
        facts: facts,
        teams_url: teams_url
      )
    end
  end

end

