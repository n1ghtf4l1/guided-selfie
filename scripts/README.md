# userkeys.mjs

CLI tool for managing user API keys in Upstash Redis.

Requires `KV_REST_API_URL` and `KV_REST_API_TOKEN` environment variables.

## Commands

### create

Create a new user key. Prints the plaintext token once.

```bash
node scripts/userkeys.mjs create --userId <id> [--status active|disabled] [--token <custom-token>]
```

### show

Show a user key record.

```bash
node scripts/userkeys.mjs show --token <token>
node scripts/userkeys.mjs show --id <tokenId>
```

### list

List all keys for a user.

```bash
node scripts/userkeys.mjs list --userId <id>
```

### disable / enable

Disable or enable a user key.

```bash
node scripts/userkeys.mjs disable --token <token>
node scripts/userkeys.mjs enable --id <tokenId>
```

### delete

Delete a user key.

```bash
node scripts/userkeys.mjs delete --token <token>
```

### rotate

Issue a new token for the same user and disable the old one.

```bash
node scripts/userkeys.mjs rotate --token <old-token>
```

### update

Update fields of a user key.

```bash
node scripts/userkeys.mjs update --token <token> --status disabled
```
