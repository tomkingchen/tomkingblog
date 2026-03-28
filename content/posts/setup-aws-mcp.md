---
title: "How to Setup Aws MCP in VSCode with AWS SSO"
date: 2026-03-28T16:00:55+11:00
draft: true
---

The official doc and Github doc only provide basic information when you use standard `aws configure` command to setup AWS Auth. But what if you use script like aws-sso-login and other bespoke scripts to login?

Below steps are based on you have already done the prerequisites steps. E.g, uv is installed.

## Restart VSCode
When using the standard configure below, I was getting `Error spawn uvx ENOENT` error.

```Json
		"aws-mcp": {
			"command": "uvx",
			"args": [
				"mcp-proxy-for-aws@latest",
				"https://aws-mcp.us-east-1.api.aws/mcp",
				"--metadata", "AWS_REGION=us-west-2"
			],
			"env": {
				"FASTMCP_LOG_LEVEL": "ERROR"
			}
		}
```

After restarting VSCode, this error no longer appears.

## Specify AWS Profile
After restarted VSCode, the MCP Server still comes up with error. But this time it is very obvious.
```Log
2026-03-27 13:58:39.666 [warning] [server stderr] ValueError: No AWS credentials found. Please configure your AWS credentials using 'aws configure' or environment variables.
```

I can see the MCP command supports `--profile` argument
 ![image.png](https://blogfilesr2.tomking.xyz/aws-mcp-cmd-args.png)

So I updated the MCP config as below.
```Json
		"aws-mcp": {
			"command": "uvx",
			"args": [
				"mcp-proxy-for-aws@latest",
				"https://aws-mcp.us-east-1.api.aws/mcp",
				"--profile", "tom",
				"--metadata", "AWS_REGION=ap-southeast-2"
			],
			"env": {
				"FASTMCP_LOG_LEVEL": "ERROR"
			}
		}
```

And sign in through my sign in script, the MCP Server was able to start normally.
 ![image.png](https://blogfilesr2.tomking.xyz/aws-mcp-start.png)

With that, now if I ask Copilot with prompt like: `Draw a diagram for network in ap-southeast-2`

This is what I can get. A clear summary for my VPC network and a nice Mermaid diagram.
 ![image.png](https://blogfilesr2.tomking.xyz/aws-mcp-answer.png)
