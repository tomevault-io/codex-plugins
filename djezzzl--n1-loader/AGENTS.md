# Copilot Instructions

Every PR must have no errors from:

```
bundle exec rubocop
bundle exec rspec spec/n1_loader_spec.rb
bundle exec rspec spec/n1_loader_spec.rb spec/activerecord_spec.rb
bundle exec rspec spec/n1_loader_spec.rb spec/activerecord_spec.rb spec/ar_lazy_preload_spec.rb
```

---
> Source: [djezzzl/n1_loader](https://github.com/djezzzl/n1_loader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-29 -->
