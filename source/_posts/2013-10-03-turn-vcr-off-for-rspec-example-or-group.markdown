---
layout: post
title: "Turn VCR off for rspec example or group"
date: 2013-10-03 01:33
comments: true
categories: 
---

I often use [VCR](https://github.com/vcr/vcr) gem.
And I always configure it to use [rspec metadata](https://www.relishapp.com/vcr/vcr/v/2-6-0/docs/test-frameworks/usage-with-rspec-metadata) to turn it on with just `:vcr` symbol. 

When I know that some module will always hit the external service,
I turn VCR on at the very top `describe`.

```ruby
describe "Something", :vcr do
  it "should make coffee" do
  end
end
```

And when I add a new example, I want all requests to be live, until I get my test pass.
I can run any code without VCR in `VCR.turned_off do ... end` block. I must eject the loaded cassette before with `VCR.eject_cassette` (there must be a reason why it can't be done automatically).

And finally since I'm using webmock (by default), I have to ask it to let me do real requests with `WebMock.allow_net_connect!`

I think it's not fair when turning something on is easy, but turning it off isn't.

I would write

```ruby
describe "Something", :vcr do
  it "should make coffee" do
  end

  it "should be easy", :vcr_off do
  end
end
```

Here's my VCR setup files (which usually goes to `spec/support` directory)

```ruby
VCR.configure do |c|
  c.cassette_library_dir = 'spec/fixtures/vcr_cassettes'
  c.hook_into :webmock # or :fakeweb
  c.ignore_localhost          = true
  c.default_cassette_options  = { :record => :new_episodes }
  c.configure_rspec_metadata!
end

RSpec.configure do |c|
  c.before :each, vcr_off: true do
    WebMock.allow_net_connect!
  end

  c.after :each, vcr_off: true do
    WebMock.disable_net_connect!
  end

  c.around :each, vcr_off: true do |ex|
    VCR.eject_cassette
    VCR.turned_off do
      ex.run
    end
  end
end
```
