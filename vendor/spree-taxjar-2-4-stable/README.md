SpreeTaxjar
===========

Spree::Taxjar is a sales tax extension for Spree using [SmartCalcs by TaxJar](https://developers.taxjar.com/api/reference/).

## Prerequisites

- Create a new account with [TaxJar](http://www.taxjar.com/)
- Go to `Account >> SmartCalcs API` to generate an API token
    - API token is needed to calculate sales tax from TaxJar over API
- Go to `Account >> State Settings` and click the [Add State with Nexus](http://blog.taxjar.com/sales-tax-nexus-definition/) button for each state where want/need to collect sales tax.
    - **NOTE:** TaxJar returns ZERO sales tax by default for orders shipping to states which are not designated as a nexus state.

Demo
-----------------------------------
Try Spree Taxjar for Spree Master with direct deployment on Heroku:

[![Deploy](https://www.herokucdn.com/deploy/button.svg)](https://heroku.com/deploy?template=https://github.com/vinsol-spree-contrib/spree-demo-heroku/tree/spree-taxjar-master)

Try Spree Taxjar for Spree 3-4 with direct deployment on Heroku:

[![Deploy](https://www.herokucdn.com/deploy/button.svg)](https://heroku.com/deploy?template=https://github.com/vinsol-spree-contrib/spree-demo-heroku/tree/spree-taxjar)

Try Spree Taxjar for Spree 3-1 with direct deployment on Heroku:

[![Deploy](https://www.herokucdn.com/deploy/button.svg)](https://heroku.com/deploy?template=https://github.com/vinsol-spree-contrib/spree-demo-heroku/tree/spree-taxjar-3-1)

 *NOTE**: Don't forget to create a taxjar account and update the key in your admin section && Update tax rates to Spree/Taxjar calculator.

## Installation

1. Add this extension to your Gemfile with this line:

  ```ruby
  gem 'spree_taxjar', github: 'vinsol-spree-contrib/spree_taxjar'
  ```

  **Note** For spree 3-1 use `3-1-stable` branch. For spree version > 3-1 use `master` branch.

2. Install the gem using Bundler:

  ```ruby
  bundle install
  ```

3. Copy & run migrations

  ```ruby
  bundle exec rails g spree_taxjar:install
  ```

4. Restart your server

  If your server was running, restart it so that it can find the assets properly.

5. Go to Admin >> Configurations >> TaxJar Settings
  - Add your TaxJar API Token
  - Check the `TAXJAR ENABLED` checkbox
  - Optionally, check `TAXJAR DEBUG ENABLED` for debugging issues
    - Not recommended for production use unless debugging production issues
  - Update tax rates to Spree/Taxjar calculator.

## Developing / Debugging Extension

- Ensure `Spree::Config[:taxjar_enabled]` is set as expected (true/false)
- Set `Spree::Config[:taxjar_debug_enabled]` as true
    - It starts logging the interactions in `spec/dummy/log/spree_taxjar.log` if using tests
    - Check the logs in your Rails application AT `log/spree_taxjar.log` where you have installed the spree_taxjar extension
    - The same logs are also added to Rails log file like `log/development.log` (works for all environments)
- As most of the API interactions are recorded and stored in VCR cassettes AT `spec/fixtures/vcr_cassettes`
    - Start with getting familiar with request and response expected
    - Feel free to delete the cassettes to debug your live use-case by setting `Spree::Config[:taxjar_api_key]` as your api_key and inspect results

## TaxJar API Usage and Billing

- We have chosen accuracy over price for obvious reasons (high fine by states if sales tax compliance breach is found)
- Minimum of N + 1 tax calculation API calls are made for each order with N shipments

TaxJar provides 2 set of API tiers (Standard and Advanced) but shipping in both cases is sent as single unit which makes Sales Tax calculation for different shipments a bit tricky or hacky.

- We chose to use both API methods at our advantage
- For line\_items sales tax, Advanced Tier API call is made to take advantage of in-built product_tax_code and discount fields and all line_items are grouped and sent in same API call
    - Shipping component is ignored as it returns tax for whole order shipping cost and not shipment wise
- For shipments, standard tier API call is made for each shipment with order amount as ZERO and shipping as shipment cost as we are only interested in shipping charge and not the whole order tax which solves the problem of inaccurate and hackish implementation.

## Testing

First bundle your dependencies, then run `rake`. `rake` will default to building the dummy app if it does not exist, then it will run specs. The dummy app can be regenerated by using `rake test_app`.

```shell
bundle
bundle exec rake
```

## Credits

[![vinsol.com: Ruby on Rails, iOS and Android developers](http://vinsol.com/vin_logo.png "Ruby on Rails, iOS and Android developers")](http://vinsol.com)

Copyright (c) 2016 [vinsol.com](http://vinsol.com "Ruby on Rails, iOS and Android developers"), released under the New MIT License

## Note

For better handling of exceptions raised by TaxJar due to various validations add the following code to your project's `application_controller.rb`:

    rescue_from Taxjar::Error, with: :taxjar_rollback

    private
      def taxjar_rollback(e)
        flash[:error] = 'TaxJar::' + e.message
        redirect_to cart_path
      end
