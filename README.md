[![Continuous Integration](https://github.com/bdurand/lumberjack_capture_device/actions/workflows/continuous_integration.yml/badge.svg)](https://github.com/bdurand/lumberjack_capture_device/actions/workflows/continuous_integration.yml)
[![Ruby Style Guide](https://img.shields.io/badge/code_style-standard-brightgreen.svg)](https://github.com/testdouble/standard)

# Lumberjack Capture Device

This is a plugin device for the [lumberjack gem](https://github.com/bdurand/lumberjack) that enables capturing log messages in a test suite so that assertions can be made against them. It provides and easy and stable method of testing that specific log messages are being sent to a logger.

Using mocks and stubs on a logger to test that it receives messages can make for a brittle test suite since there can a wide variety of code writing messages to logs and your test suite may have a higher log level turned on causing it skip messages at a lower level.

For instance, this rspec code can break if any of the code called by the `do_something` writes a different info log message:

```ruby
do_something
expect(Rail.logger).to receive(:info).with("Something happened")
```

It will also break if the test suite logger has the log level set to `warn` or higher since it will then skip all info and debug messages.

## Usage

You can call the `Lumberjack::CaptureDevice.capture` method to override a logger so that it will capture log entries within a block to an in memory buffer. This method will yield the capturing log device as well as return it as the result of the method. The log level will also be temporarily set to debug within the block, so you can capture all log messages without having to change the log level for the entire test suite.

You can use the `include?` method on the log device to determine if specific log entries were made. This would be the equivalent code to the above rspec test, but without the brittleness of mocking method calls:

```ruby
Lumberjack::CaptureDevice.capture(Rails.logger) do |logs|
  do_something
  expect(logs).to include(level: :info, message: "Something happened")
end
```

You can also write that same test as:

```ruby
logs = Lumberjack::CaptureDevice.capture(Rails.logger) { do_something }
expect(logs).to include(level: :info, message: "Something happened")
```

You can also use the `Lumberjack::CaptureDevice#extract` method with the same arguments as used by `include?` to grab all lines that match the filters. And finally, you can access all the log entries with `Lumberjack::CaptureDevice#buffer`.
