require 'bundler'
Bundler.require

require 'opal/rspec/rake_task'
Opal::RSpec::RakeTask.new(:default)

desc "Build opal/opal/rspec/rspec.js"
task :build do
  File.open('opal/opal/rspec/rspec.js', 'w+') do |out|
    out << build_rspec
  end
end

desc "Show dev/min sizes"
task :sizes do
  code = build_rspec
  min  = uglify code

  puts "\ndevelopment: #{code.size}, minified: #{min.size}"
end

module Opal
  module Nodes
    class OpAsgn1Node
      def compile_and
        with_temp do |a| # args
          with_temp do |r| # recv
            aref = s(:call, s(:js_tmp, r), :[], s(:arglist, s(:js_tmp, a)))
            aset = s(:call, s(:js_tmp, r), :[]=, s(:arglist, s(:js_tmp, a), rhs))
            andop = s(:and, aref, aset)

            push "(#{a} = ", expr(first_arg), ", #{r} = ", expr(lhs)
            push ", ", expr(andop), ")"
          end
        end
      end
    end
  end
end

def build_rspec
  Opal::Processor.dynamic_require_severity = :warning

  code = []
  gems = %w(rspec rspec-core rspec-support rspec-expectations rspec-mocks)

  gems.each do |gem_name|
    spec = Gem::Specification.find_by_name gem_name
    gem_dir = File.join spec.gem_dir, 'lib'
    prefix = gem_dir + '/'

    Dir.glob(File.join(gem_dir, '**/*.rb')).each do |source|
      requirable = source.sub(prefix, '').sub(/\.rb$/, '')

      compiler = Opal::Compiler.new File.read(source),
        requirable: true, file: requirable, dynamic_require_severity: :warning

      code << compiler.compile
    end
  end

  stubs = %w(shellwords fileutils optparse)

  stubs.each do |stub|
    compiler = Opal::Compiler.new '', requirable: true, file: stub
    code << compiler.compile
  end

  code.join "\n"
end

def uglify(str)
  IO.popen('uglifyjs', 'r+') do |i|
    i.puts str
    i.close_write
    return i.read
  end
rescue Errno::ENOENT
  $stderr.puts '"uglifyjs" command not found (install with: "npm install -g uglify-js")'
  nil
end
