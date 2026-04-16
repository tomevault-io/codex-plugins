# Platform
- Use RSpec when building tests

## Spec Structure
Use FactoryBot for factories and configure necessary helpers:

```ruby
RSpec.describe UserServices::Disable, type: :service do
  subject(:service) { described_class.new(user) }

  let(:user) { create(:user, valid:) }
  let(:valid) { false }

  describe '#call' do
    subject { service.call }

    context 'when user is valid' do
      let(:valid) { true }

      it 'disables user successfully' do
        expect(subject).to be_success
        expect(user.reload).to be_disabled
      end
    end
  end
end
```

### Test Helpers
- Use `FactoryBot::Syntax::Methods` for `create`, `build`
- Configure `Devise::Test::ControllerHelpers` for controller tests

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/eventaservo)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/eventaservo)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
