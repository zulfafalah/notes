

1. Buka/Buat file configurasi
```
nano ~/.config/opencode/opencode.json
```
2. Masukan configurasi dibawah ini
```
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "SumoPod": {
      "npm": "@ai-sdk/openai-compatible",
      "options": {
        "baseURL": "https://ai.sumopod.com/v1"
      },
       "models": {
        "kimi-k2-250905": {},
        "azure_ai/kimi-k2.5": {},
        "glm-4-7-251222": {},
        "qwen3.6-plus": {},
        "claude-sonnet-4-6": {},
        "deepseek-v3-2": {},
        "kimi-k2.6" :{},
        "deepseek-v4-pro" :{}
      }
    }
  }
}
```

3. Refresh models
```
opencode models --refresh
```
