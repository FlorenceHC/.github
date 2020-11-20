# frozen_string_literal: true

@ci_files = %w[Gemfile Brewfile Makefile travis.yml Dangerfile]

# determine if any of the files were modified
def did_modify(files_array)
  did_modify_files = false
  files_array.each do |file_name|
    next unless git.modified_files.include?(file_name) || git.deleted_files.include?(file_name)

    did_modify_files = true
    config_files = git.modified_files.select { |path| path.include? file_name }
    message "This PR changes #{github.html_link(config_files)}"
  end

  did_modify_files
end

###
### Warnings
###

warn('Changes to CI/CD files') if did_modify(@ci_files)

# Sometimes it's a README fix, or something similar, which isn't relevant for
# including in a project's CHANGELOG for example
not_declared_trivial = !(github.pr_title.downcase.include? '#trivial')

warn('PR is classed as Work in Progress') if github.pr_title.include? '[WIP]'

warn('Big PR, try to keep changes smaller if you can') if git.lines_of_code > 250

# Milestones are required for all PRs to track what's included in each release
has_milestone = !github.pr_json['milestone'].nil?

warn('This PR is missing a milestone... please add one 😉', sticky: false) unless has_milestone

has_source_changes = !git.modified_files.grep(/workflow-templates/).empty?

if has_source_changes && !not_declared_trivial
  warn('Any major changes should be reflected in the Changelog.
			Please consider adding a note there and adhere to the
			[Changelog Guidelines](https://keepachangelog.com/en/1.0.0/).')
end

# Look through all changed Markdown files
markdown_files = (git.added_files.grep(%r{.*\.md/}) + git.modified_files.grep(%r{.*\.md/}))

unless markdown_files.empty?
  # Run proselint to lint and check spelling
  prose.language = 'en-us'
  prose.ignored_words = %w[dependabot]
  prose.ignore_acronyms = true
  prose.ignore_numbers = true
  prose.lint_files markdown_files
  prose.check_spelling markdown_files
end

jira.check(
  key: %w[MOB IOS EC],
  url: 'https://florencehc.atlassian.net/browse',
  search_title: true,
  search_commits: false,
  fail_on_warning: false,
  report_missing: true,
  skippable: true
)
