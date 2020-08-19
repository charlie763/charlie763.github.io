---
layout: post
title:      "Trying Test Driven Development For the First Time with Rspec and Capybara"
date:       2020-08-19 23:15:56 +0000
permalink:  trying_test_driven_development_for_the_first_time_with_rspec_and_capybara
---


As a former product manager, I would hear my dev teams talk about TDD, but I didn’t have a great understanding of what it’s about. As a student at the Flatiron Software Engineering program, I’ve gotten a good amount of practice interacting with tests. Each of the coding labs we do has a suite of tests we have to make pass. But, those test are written by other people. So, I wanted to use my Rails portfolio project as an opportunity to practice TDD and writing tests in general.

I would say that my efforts at TDD were a moderate success. The primary issue has been time. Writing test, particularly writing tests for the first time, takes time. We’re given a limited amount of time to complete our portfolio projects, and it’s not required that we write tests. So, I started out by doing real TDD, but it eventually became an effort to simply become familiar with Rspec and Capybara. My test coverage for my portfolio project is incomplete, but I was able to get a better sense of TDD and how to write tests generally. Here are the spark notes of what I learned:

General takeaways
    • Writing tests takes time.
    • But those tests helped me identify bugs quicker. I can see how tests save time in the long term, particularly as applications become more complex.
    • Even though I don’t have great test coverage, I have more confidence that when I’m refactoring or adding new features later, I’ll know if I accidentally break something else. AND, I don’t have to click all over my website repetitively to get that confidence.
    • Writing tests helped me think through my design choices, and made it easier to see some features that weren’t needed.

Gotchas
    • Rspec + Rails expects you to write different types of tests that correspond roughly to the MVC convention. For example, you can’t access a session from a controller type test; you’d need to write a feature type test.
    • To access sessions from a test you need to add ` config.middleware.use RackSessionAccess::Middleware` to your `config/environment/test.rb` file
    • You can access seed data from a test, but you need to add ` Rails.application.load_seed` to your `rails_helper.rb` file to do so.

All in all, writing tests was a good learning experience. In terms of developing my app, I was able to quickly identify bugs that lead to learning. For example, when one test failed, I learned that it was because the `belongs to` macro for Active Records associations makes it so that that association is required by default. You can change this by adding the option `required: false` to the macro. In addition, Capybara testing helped me understand an important distinction about the `form_with` Form Helper. In this case, my test was passing, but the behavior on the website wasn’t as expected. It turns out that the `form_with` helper (as opposed to the `form_for` and `form_tag` helpers I had used in Flatiron labs) submits your form using javascript by default. If you’re rendering a template at the end of a controller action instead of redirecting to another page (i.e. so you can easily display errors for a create action) this creates problems. The template gets rendered on the backend but isn’t necessarily sent back to the browser. This can be fixed  by reverting to `form_tag` or setting `local: true` as an option in your `form_with` tag.

Without writing tests, debugging may have been significantly harder. I’m not sure if I’m 100% sold on TDD, but I definitely see the value of writing tests in general.
