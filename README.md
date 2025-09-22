# Gemini Code-Assist Tunnel + Proxy

**Purpose (end user)**  
This workflow makes Google Gemini Code Assist available locally on a remote server by creating a secure SSH tunnel and a small user-space proxy. After running the workflow, point your Code Assist provider (for example, Cline in VS Code) at:

**Final client URL:** `http://127.0.0.1:25000`

Use your normal Gemini API key with the provider.

---

## What this does (high level)
1. Creates an SSH tunnel from the remote host to `generativelanguage.googleapis.com:443`.  
2. Runs a local proxy on the remote host that listens on `127.0.0.1:25000`.  
3. The proxy forwards traffic through the SSH tunnel to Google so your Code Assist client can call Gemini as if it were local.

You do **not** need root on the remote host. The proxy runs in a Python virtual environment (venv) so it’s portable and repeatable.

---

## How to connect (end-user steps)
1. Run the ACTIVATE workflow on the chosen remote resource. The workflow will:
   - open the SSH tunnel (default local tunnel port `8443`), and
   - start the Python/mitmproxy-based proxy (default local proxy port `25000`).

2. In your Code Assist provider (Cline in VS Code), set:
   - **Base URL**: `http://127.0.0.1:25000`
   - **API key**: your usual Google API key

3. Use Code Assist as you normally would; requests will flow through the proxy and tunnel to Gemini.

---

## Important notes for end users
- **URL to use in Cline / VS Code:** `http://127.0.0.1:25000`  
  (This is the final, stable endpoint the workflow exposes for your code-assist provider.)
- **API key:** continue to use your existing Google API key in the provider settings.
- **TLS/CAs:** The workflow uses mitmproxy (in a venv). For most setups, the workflow arranges TLS proxying automatically. If your client shows TLS errors, you may need to add the generated `mitmproxy` CA to the client trust store. Contact your admin if you cannot add custom CAs.
- **No root required:** Everything runs in user space (venv, mitmproxy, SSH tunnel).
- **Security:** Do not share your API key or private SSH keys in public repos. The workflow is intended for safe, temporary testing — tear it down when not in use.

---

## Help and troubleshooting (quick tips)
- If Code Assist shows connection errors, verify the proxy is listening on `127.0.0.1:25000`.
- If TLS errors appear, the client may not trust the proxy CA — try restarting VS Code with the mitm CA trusted, or ask your admin for help.
- If you see `404 Not Found`, that usually means the API endpoint/path was incorrect — `404` still indicates the tunnel/proxy is functioning.
