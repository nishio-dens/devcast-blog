---
title: "Railsの挙動を確認するときに使いたいコードテンプレート"
author: nishio-dens
categories: Rails
tags: ActiveRecord
draft: false
published_at: 2018-02-13 18:00
---

Railsの挙動を確認したいとき、いちいち ``rails new`` コマンドでプロジェクトを作り、
DB migrationファイルやモデルを作って動作確認するのは面倒です。

今回はRailsの簡単な動作確認に使えるコードスニペットを紹介します。

<!-- more -->


Rails開発チームは、バグレポート用のテンプレートコードを以下に用意しています。

- https://github.com/rails/rails/tree/master/guides/bug_report_templates

このテンプレートは手元でRailsの挙動を動作確認する際にも利用できます。

# ActiveRecordの動作確認用コード(sqlite3)

ActiveRecordの挙動を確認したい場合は、以下テンプレートを使います。

- RailsリリースバージョンのActiveRecordテンプレート
  - https://github.com/rails/rails/blob/master/guides/bug_report_templates/active_record_gem.rb
- Rails masterバージョンのActiveRecordテンプレート
  - https://github.com/rails/rails/blob/master/guides/bug_report_templates/active_record_master.rb

active_record_gem.rb は以下のようになっています。

```ruby
# frozen_string_literal: true

begin
  require "bundler/inline"
rescue LoadError => e
  $stderr.puts "Bundler version 1.10 or later is required. Please update your Bundler"
  raise e
end

gemfile(true) do
  source "https://rubygems.org"

  git_source(:github) { |repo| "https://github.com/#{repo}.git" }

  # ここに使いたいgemを書きます
  gem "activerecord", "5.1.0"
  gem "sqlite3"
end

require "active_record"
require "minitest/autorun"
require "logger"

# Ensure backward compatibility with Minitest 4
Minitest::Test = MiniTest::Unit::TestCase unless defined?(Minitest::Test)

# Sqliteの場合は以下のようにしてインメモリDBを用意します
ActiveRecord::Base.establish_connection(adapter: "sqlite3", database: ":memory:")
ActiveRecord::Base.logger = Logger.new(STDOUT)

ActiveRecord::Schema.define do
  # DB Schema定義を書きます
  create_table :posts, force: true do |t|
  end

  create_table :comments, force: true do |t|
    t.integer :post_id
  end
end

class Post < ActiveRecord::Base
  has_many :comments
end

class Comment < ActiveRecord::Base
  belongs_to :post
end

class BugTest < Minitest::Test
  def test_association_stuff
    # テストコードを書きます
    # pryをインストールして、binding.pry で動作確認などするとよいかもしれません
    post = Post.create!
    post.comments << Comment.create!

    assert_equal 1, post.comments.count
    assert_equal 1, Comment.count
    assert_equal post.id, Comment.first.post.id
  end
end
```


DB のマイグレーションコードもテストしたい場合は、
https://github.com/rails/rails/blob/master/guides/bug_report_templates/active_record_migrations_gem.rb のテンプレートを利用します。

# ActiveRecordの動作確認用コード(mysql)

mysqlの場合は以下のようにしてDBの生成をします。

```ruby
# frozen_string_literal: true

begin
  require "bundler/inline"
rescue LoadError => e
  $stderr.puts "Bundler version 1.10 or later is required. Please update your Bundler"
  raise e
end

gemfile(true) do
  source "https://rubygems.org"

  git_source(:github) { |repo| "https://github.com/#{repo}.git" }

  gem "activerecord", "5.1.0"
  # mysql2をインストールします
  gem "mysql2"
end

require "active_record"
require "minitest/autorun"
require "logger"

Minitest::Test = MiniTest::Unit::TestCase unless defined?(Minitest::Test)

# 実際にmysqlのDBを作成して動作確認をします
db_config_admin = {
  adapter: "mysql2",
  username: ENV["USERNAME"] || "root",
  password: ENV["PASSWORD"] || "",
  host: ENV["HOST"] || "localhost"
}
db_config = db_config_admin.merge(
  database: "rails_test_database"
)

ActiveRecord::Base.establish_connection db_config_admin
ActiveRecord::Base.connection.drop_database db_config[:database] rescue nil
ActiveRecord::Base.connection.create_database db_config[:database]
ActiveRecord::Base.connection.close
ActiveRecord::Base.establish_connection db_config
ActiveRecord::Base.logger = Logger.new(STDOUT)

ActiveRecord::Schema.define do
  # DB Schema定義を書きます
  create_table :posts, force: true do |t|
  end

  create_table :comments, force: true do |t|
    t.integer :post_id
  end
end

class Post < ActiveRecord::Base
  has_many :comments
end

class Comment < ActiveRecord::Base
  belongs_to :post
end

class BugTest < Minitest::Test
  def test_association_stuff
    # テストコードを書きます
    post = Post.create!
    post.comments << Comment.create!

    assert_equal 1, post.comments.count
    assert_equal 1, Comment.count
    assert_equal post.id, Comment.first.post.id
  end
end
```

# その他コードテンプレート

ActiveRecord以外にもRailsは以下バグレポート用のテンプレートを用意しています。

* Action Controller
  * https://github.com/rails/rails/blob/master/guides/bug_report_templates/action_controller_gem.rb
* Active Job
  * https://github.com/rails/rails/blob/master/guides/bug_report_templates/active_job_gem.rb
* その他汎用的なテンプレート
  * https://github.com/rails/rails/blob/master/guides/bug_report_templates/generic_gem.rb
