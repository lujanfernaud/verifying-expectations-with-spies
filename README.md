# Upcase: Test Doubles

## Verifying Expectations with Spies

Verifying expectations with spies for the [Test Doubles](https://thoughtbot.com/upcase/test-doubles) course.

### Before

```ruby
# signup_spec.rb

describe Signup do
  describe "#save" do
    it "creates an account with one user" do
      signup = Signup.new(email: "user@example.com", account_name: "Example")

      result = signup.save

      expect(Account.count).to eq(1)
      expect(Account.last.name).to eq("Example")
      expect(User.count).to eq(1)
      expect(User.last.email).to eq("user@example.com")
      expect(User.last.account).to eq(Account.last)
      expect(result).to be(true)
    end
  end

  describe "#user" do
    it "returns the user created by #save" do
      signup = Signup.new(email: "user@example.com", account_name: "Example")
      signup.save

      result = signup.user

      expect(result.email).to eq("user@example.com")
      expect(result.account.name).to eq("Example")
    end
  end
end
```

### After

```ruby
# signup_spec.rb

describe Signup do
  describe "#save" do
    it "creates an account with one user" do
      email   = "user@example.com"
      account = double("account", name: "Example")

      stub_account_creation_with(account)
      stub_user_creation_with(account, email)

      signup = Signup.new(email: email, account_name: account.name)
      result = signup.save

      expect(Account).to have_received(:create!).with(name: account.name)
      expect(User).to have_received(:create!).with(account: account, email: email)
      expect(result).to be(true)
    end
  end

  describe "#user" do
    it "returns the user created by #save" do
      email   = "user@example.com"
      account = double("account", name: "Example")
      user    = double("user", account: account, email: email)

      stub_account_creation_with(account)
      stub_user_creation_with(account, email).and_return(user)

      signup = Signup.new(email: email, account_name: account.name)
      signup.save

      result = signup.user

      expect(result.email).to eq("user@example.com")
      expect(result.account.name).to eq("Example")
    end
  end

  def stub_account_creation_with(account)
    allow(Account).to receive(:create!)
      .with(name: account.name)
      .and_return(account)
  end

  def stub_user_creation_with(account, email)
    allow(User).to receive(:create!)
      .with(account: account, email: email)
  end
end
```
