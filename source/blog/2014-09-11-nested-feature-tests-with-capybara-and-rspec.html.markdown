---
title: "Nested feature tests with Capybara and RSpec"
date: 2014-09-11 15:46:00 -0400
tags: Rspec, Capybara, Ruby On Rails, testing
---

I recently wrote some automated user acceptance tests with [Capybara](https://github.com/jnicklas/capybara) for a multi-step wizard, and the initial approach didn't seem very DRY. I was rehashing the steps involved in previous scenarios to reach whatever step was currently being tested, and I was combining multiple expectations at various steps:

``` ruby
feature 'User creates a thing' do
  Capybara.javascript_driver = :poltergeist
  WebMock.disable_net_connect!(:allow_localhost => true)

  background do
    user_login
  end

  scenario 'Step 0: User signs in' do
    expect(page).to have_content('Signed in successfully.')
  end

	scenario 'Step 1: selects A, continues, reaches Step 2', :js => true do
    visit('/wizard/type')
    expect(page).to have_content('Step 1')
    find('.type-a').click
    find('.continue').click
    expect(page).to have_content('Step 2')
	end

	scenario 'Step 2: picks name, continues, reaches Step 3', :js => true do
    visit('/wizard/type')
    find('.type-a').click
    find('.continue').click
    expect(page).to have_content('Step 2')

    fill_in 'name', with: 'Funeralopolis'
    find('.continue').click
    expect(page).to have_content('Step 3')
	end
end
```

<br>
Thankfully you can [nest features since the 2.2.1 version](https://github.com/jnicklas/capybara/commit/61524b0fd32a1fb55b82853bdc4a0293e9fcdef0), which allows you to better separate the tests and setups while sequentially continuing from whatever state the previous test left you in:
``` ruby
feature 'User creates a thing' do
  Capybara.javascript_driver = :poltergeist
  WebMock.disable_net_connect!(:allow_localhost => true)

  background do
    user_login
  end

  scenario 'Step 0: User signs in' do
    expect(page).to have_content('Signed in successfully.')
  end

  feature 'Step 1: Type', :js => true do
    background do
      visit('/wizard/type')
    end

    scenario 'it is on Step 1' do
      expect(page).to have_content('Step 1')
    end

    feature 'it selects A, continues, reaches Step 2' do
      background do
        find('.type-a').click
        find('.continue').click
      end

      scenario 'it reaches Step 2' do
        expect(page).to have_content('Step 2')
      end

      feature 'Step 2: Name', :js => true do
        background do
          fill_in 'name', with: 'Funeralopolis'
          find('.continue').click
        end

        scenario 'it reaches Step 3' do
          expect(page).to have_content('Step 3')
        end
      end
    end
  end
end
```
