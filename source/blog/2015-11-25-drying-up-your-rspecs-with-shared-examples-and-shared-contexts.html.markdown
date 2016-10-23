---
title: "DRYing up your Rspecs with shared examples and shared contexts"
date: 2015-11-25 18:35:22 -0500
tags: Rspec, testing
---

This is an example of how to use [shared contexts](https://www.relishapp.com/rspec/rspec-core/docs/example-groups/shared-context) and [shared examples](https://www.relishapp.com/rspec/rspec-core/v/3-4/docs/example-groups/shared-examples) to refactor your [specs](http://rspec.info/).

We have a Merchant model and a User model. We want to test 3 instance methods on Merchant that:

* change the merchant's state
* fire off an associated email

Let's take the most naive approach and write 3 individual tests for these methods:

``` ruby
describe '#block' do
  let(:merchant) { build(:merchant) }
  let(:billing_manager) { create(:user, :manager, merchant: merchant) }

  before do
    merchant.billing_user = billing_manager
  end

  it 'sets the merchant state to blocked' do
    merchant.block
    expect(merchant.blocked?).to be true
  end

  it 'send an email notification' do
    expect {
      merchant.block
    }.to change(ActionMailer::Base.deliveries, :count).by(1)
  end
end

describe '#unblock' do
  let(:merchant) { build(:merchant) }
  let(:billing_manager) { create(:user, :manager, merchant: merchant) }

  before do
    # only a blocked user can be unblocked
    merchant.block
    merchant.billing_user = billing_manager
  end

  it 'sets the merchant state to active' do
    merchant.unblock
    expect(merchant.active?).to be true
  end

  it 'send an email notification' do
    expect {
      merchant.unblock
    }.to change(ActionMailer::Base.deliveries, :count).by(1)
  end
end

describe '#deactivate' do
  let(:merchant) { build(:merchant) }
  let(:billing_manager) { create(:user, :manager, merchant: merchant) }

  before do
    merchant.billing_user = billing_manager
  end

  it 'sets the merchant state to deactivated' do
    merchant.deactivate
    expect(merchant.deactivated?).to be true
  end

  it 'send an email notification' do
    expect {
      merchant.deactivate
    }.to change(ActionMailer::Base.deliveries, :count).by(1)
  end
end
```

You're testing the same things on all 3 methods, so you can leverage shared examples to rewrite that in a way where you don't repeat that. You also want to pass both the action to trigger and the new state to test as parameters to your shared example:

``` ruby
shared_examples_for 'a merchant state change' do |action, new_state|
  it "sets the merchant state to #{new_state}" do
    merchant.send(action)
    expect(merchant.send("#{new_state}?")).to be true
  end

  it 'sends an email notification' do
    expect {
      merchant.send(action)
    }.to change(ActionMailer::Base.deliveries, :count).by(1)
  end
end

describe '#block' do
  let(:merchant) { build(:merchant) }
  let(:billing_manager) { create(:user, :manager, merchant: merchant) }

  before do
    merchant.billing_user = billing_manager
  end

  it_behaves_like 'a merchant state change', 'block', 'blocked'
end

describe '#unblock' do
  let(:merchant) { build(:merchant) }
  let(:billing_manager) { create(:user, :manager, merchant: merchant) }

  before do
    merchant.billing_user = billing_manager
    merchant.block
  end

  it_behaves_like 'a merchant state change', 'unblock', 'active'
end

describe '#deactivate' do
  let(:merchant) { build(:merchant) }
  let(:billing_manager) { create(:user, :manager, merchant: merchant) }

  before do
    merchant.billing_user = billing_manager
  end

  it_behaves_like 'a merchant state change', 'deactivate', 'deactivated'
end
```

Now that you've done that, you want to avoid repeating the setup portion for these tests. You can do that with [shared contexts](https://www.relishapp.com/rspec/rspec-core/docs/example-groups/shared-context):

``` ruby
shared_examples_for 'a merchant state change' do |action, new_state|
  it "sets the merchant state to #{new_state}" do
    merchant.send(action)
    expect(merchant.send("#{new_state}?")).to be true
  end

  it 'sends an email notification' do
    expect {
      merchant.send(action)
    }.to change(ActionMailer::Base.deliveries, :count).by(1)
  end
end

shared_context 'has a merchant and a billing manager' do
  let(:merchant) { build(:merchant) }
  let(:billing_manager) { create(:user, :manager, merchant: merchant) }

  before do
    merchant.billing_user = billing_manager
  end
end

describe '#block' do
  include_context 'has a merchant and a billing manager'
  it_behaves_like 'a merchant state change', 'block', 'blocked'
end

describe '#unblock' do
  include_context 'has a merchant and a billing manager'
  before do
    merchant.block
  end

  it_behaves_like 'a merchant state change', 'unblock', 'active'
end

describe '#deactivate' do
  include_context 'has a merchant and a billing manager'
  it_behaves_like 'a merchant state change', 'deactivate', 'deactivated'
end
```

You're now down from 61 lines to 40, and it just looks less copy-paste happy. You can also move out your shared contexts to separate files and reuse them in different specs. The only immediate downside is that the line numbering is a bit less informative with shared examples compared to distinct specs.

Like anything DRY-related, use this responsibly and don't create a nightmare bizarro world of obfuscated tests.
