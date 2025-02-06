---
layout: post
title: Secure Your Secrets with Rails Credentials
date: 2025-02-06 16:51 +0300
categories: [Development]
tags: [rails]
image:
  path: https://images.unsplash.com/photo-1508345228704-935cc84bf5e2
---
Hello 👋

Whether you are a junior or senior developer, you know that keeping secrets out of your repository is crucial. From API keys to database passwords, sensitive information should never be hardcoded into your application. Traditionally, developers have used environment variables, the `dotenv` gem, or YAML-based configurations like `secrets.yml` to manage secrets. While these methods work, they come with their own challenges—environment variables can be hard to manage across teams, and `.env` files need to be excluded from version control manually.

Rails provides a built-in way to manage secrets securely using rails credentials. In this post, we'll explore what credentials are, how to use them effectively, and some best practices to keep your application secure.

## What Are Rails Credentials?
Introduced in Rails 5.2, rails credentials is an encrypted file-based system for storing sensitive configuration data. Unlike `.env` files or `secrets.yml`, Rails credentials are encrypted and can be safely checked into your repository.

By default, Rails provides a credentials file located at `config/credentials.yml.enc`, which is encrypted using a key stored in `config/master.key` (or `ENV['RAILS_MASTER_KEY']` in production environments). This ensures that your secrets remain safe and encrypted in version control.

## Managing Credentials
To edit credentials, use the following command:

```bash
$ rails credentials:edit
```

This will open the encrypted file in your lovely editor. If you haven't set up an editor, you can specify one explicitly:

```bash
$ EDITOR="code --wait" rails credentials:edit
```

When you run this command for the first time, Rails will generate the `config/master.key` and an encrypted `config/credentials.yml.enc` file. The `master.key` should never be committed to version control. Instead, share it securely with your team (e.g., using a secret management tool).

It looks something like this

```yaml
aws:
  access_key_id: ABCDEFG1234567890
  secret_access_key: XYZ1234567890

secret_key_base: 1234567890abcdef1234567890abcdef
```

## Accessing Credentials
There are two way to fetch credentials from `Rails.secrets`

```ruby
Rails.application.credentials.dig(:aws, :access_key_id)
```

```ruby
Rails.application.credentials.secret_key_base
```

If you want to update credentials, you should run `EDITOR="code --wait" rails credentials:edit` again. After write down your new credentials here, close the editor tab where credentials is open.

## For Different Environments
Starting with Rails 6, you can maintain environment-specific credentials. Instead of a single `credentials.yml.enc`, you can have separate files for each environment:

```bash
$ EDITOR="code --wait" rails credentials:edit --production
```
This creates `config/credentials/production.yml.enc` and ensures credentials are separated per environment.

## Best Practices

1. 🚨 Never Commit `config/master.key`. Always add it to `.gitignore` and share it securely.

2. Use Environment-Specific Credentials – This keeps production secrets separate from development.

3. Restrict Access – Limit access to credentials and rotate keys regularly.

4. Use a Secret Management Tool – In larger applications, consider using AWS Secrets Manager, HashiCorp Vault, or similar tools.

5. Encrypt Credentials Securely – If you're using Docker, CI/CD, or Kubernetes, ensure `RAILS_MASTER_KEY` is set correctly.

## Conclusion
Rails credentials offer a secure and convenient way to manage sensitive data within your application. By understanding how to edit, store, and access credentials properly, you can enhance the security of your Rails apps while keeping your secrets safe from accidental exposure.

If you haven't already, start using Rails credentials today and keep your application secure!

Love ❤️
