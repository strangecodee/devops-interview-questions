# DevOps Interview Questions Repository

Welcome to the comprehensive DevOps interview preparation repository! This project automatically generates fresh DevOps content daily to help you prepare for interviews and stay updated with industry trends.

## 🏆 Current Stats

![Updated](https://img.shields.io/badge/Updated-2026-01-10-brightgreen)
![Interview](https://img.shields.io/badge/Interview_45-purple)
![Docker](https://img.shields.io/badge/Docker_45-blue)
![Kubernetes](https://img.shields.io/badge/K8s_46-green)
![CI/CD](https://img.shields.io/badge/CICD_46-orange)

*Last Updated: Sat Jan 10 02:26:55 UTC 2026*name: Daily DevOps Content Generator (HuggingFace)

on:
  schedule:
    - cron: "0 0 * * *"
  workflow_dispatch:

jobs:
  generate:
    runs-on: ubuntu-latest
    permissions:
      contents: write

    steps:
      # ───────────────────────────────────────────────
      # CHECKOUT
      # ───────────────────────────────────────────────
      - name: Checkout repository
        uses: actions/checkout@v3
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          fetch-depth: 0

      # ───────────────────────────────────────────────
      # DEPENDENCIES
      # ───────────────────────────────────────────────
      - name: Install dependencies
        run: sudo apt-get install -y curl jq

      # ───────────────────────────────────────────────
      # CONTENT GENERATOR (OUTPUT FIXED)
      # ───────────────────────────────────────────────
      - name: Generate DevOps content
        id: generator
        env:
          HF_TOKEN: ${{ secrets.HF_TOKEN }}
        run: |
          set -e
          export TZ="Asia/Kolkata"

          # ───── RANDOMIZATION ENGINE ─────
          RANDOM_TOPIC=$(
            printf "%s\n" \
            AWS Terraform Helm GitOps ArgoCD Istio SRE Scalability Monitoring \
            "Load Balancing" "Zero-Downtime Deploys" "Kubernetes Security" \
            "Docker Layers" "Chaos Engineering" | shuf -n 1
          )

          RANDOM_SEED=$(date +%s)

          STYLE=$(
            printf "%s\n" \
            "Enterprise tone" \
            "SRE-focused" \
            "Security-heavy" \
            "Performance-optimized" \
            "Beginner-friendly" \
            "Short and concise" | shuf -n 1
          )

          DEPTH=$(
            printf "%s\n" \
            "Low detail" \
            "Medium detail" \
            "High detail" | shuf -n 1
          )

          UNIQUE_KEYWORD=$(
            printf "%s\n" \
            phoenix nebula quantum atlas vector eclipse forge pulse matrix sentinel \
            | shuf -n 1
          )

          # ✅ EXPORT OUTPUTS IMMEDIATELY (CRITICAL)
          {
            echo "topic=$RANDOM_TOPIC"
            echo "style=$STYLE"
            echo "depth=$DEPTH"
            echo "keyword=$UNIQUE_KEYWORD"
            echo "seed=$RANDOM_SEED"
          } >> "$GITHUB_OUTPUT"

          echo "Topic: $RANDOM_TOPIC"
          echo "Style: $STYLE"
          echo "Depth: $DEPTH"
          echo "Keyword: $UNIQUE_KEYWORD"

          categories_list="devops-interview docker kubernetes cicd"

          for key in ${categories_list}; do
            prompt_file="prompts/${key}.txt"

            # ✅ Prevent crash if file missing
            [ -f "$prompt_file" ] || continue

            base_prompt=$(cat "$prompt_file")

            # Format the prompt as a chat message
            full_prompt="${base_prompt}
            Style --> ${STYLE}
            Depth --> ${DEPTH}
            UniqueKeyword --> ${UNIQUE_KEYWORD}
            Topic --> ${RANDOM_TOPIC}
            Seed --> ${RANDOM_SEED}
            "

            mkdir -p "$key"
            file="$key/$(date +'%Y-%m-%d_%H-%M').md"

            # Use Hugging Face Chat Completions API
            if [ -n "$HF_TOKEN" ]; then
              response=$(curl -s --fail \
                -X POST \
                -H "Authorization: Bearer $HF_TOKEN" \
                -H "Content-Type: application/json" \
                -d "{
                  \"messages\": [
                    {
                      \"role\": \"user\",
                      \"content\": \"$full_prompt\"
                    }
                  ],
                  \"model\": \"mistralai/Mistral-7B-Instruct-v0.2\",
                  \"stream\": false,
                  \"max_tokens\": 800,
                  \"temperature\": 0.7
                }" \
                "https://router.huggingface.co/v1/chat/completions" \
                || echo '{"error": "API response unavailable"}')
              
              # Extract the generated text from the response
              if echo "$response" | jq -e .error >/dev/null 2>&1; then
                output="API response unavailable"
              else
                output=$(echo "$response" | jq -r '.choices[0].message.content // "API response unavailable"')
              fi
            else
              output="API response unavailable - HF_TOKEN not set"
            fi

            echo "# ${key} — $(date)" > "$file"
            echo "**Topic:** ${RANDOM_TOPIC}" >> "$file"
            echo "**Style:** ${STYLE}" >> "$file"
            echo "**Depth:** ${DEPTH}" >> "$file"
            echo "**UniqueKeyword:** ${UNIQUE_KEYWORD}" >> "$file"
            echo "" >> "$file"
            echo "$output" >> "$file"
          done
        

      # ───────────────────────────────────────────────
      # VERIFY OUTPUTS (NO MORE NULL)
      # ───────────────────────────────────────────────
      - name: Verify outputs
        run: |
          echo "Topic   → ${{ steps.generator.outputs.topic }}"
          echo "Style   → ${{ steps.generator.outputs.style }}"
          echo "Depth   → ${{ steps.generator.outputs.depth }}"
          echo "Keyword → ${{ steps.generator.outputs.keyword }}"

      # ───────────────────────────────────────────────
      # JOB SUMMARY (CLEAN UI)
      # ───────────────────────────────────────────────
      - name: Workflow Summary
        run: |
          {
            echo "## Daily DevOps Content Generator"
            echo ""
            echo "- **Topic:** ${{ steps.generator.outputs.topic }}"
            echo "- **Style:** ${{ steps.generator.outputs.style }}"
            echo "- **Depth:** ${{ steps.generator.outputs.depth }}"
            echo "- **Keyword:** ${{ steps.generator.outputs.keyword }}"
          } >> $GITHUB_STEP_SUMMARY

      # ───────────────────────────────────────────────
      # GIT CONFIG
      # ───────────────────────────────────────────────
      - name: Configure Git
        run: |
          git config --global user.name "DevOps Content Bot"
          git config --global user.email "bot@github.com"
          git remote set-url origin https://github-actions:${{ secrets.GITHUB_TOKEN }}@github.com/${{ github.repository }}

      # ───────────────────────────────────────────────
      # COMMIT & PUSH
      # ───────────────────────────────────────────────
      - name: Commit and push changes
        run: |
          export TZ="Asia/Kolkata"
          git fetch origin
          git checkout main
          git add .
          if ! git diff --cached --quiet; then
            git commit -m "content: auto-generated $(date '+%Y-%m-%d %H:%M IST')"
            git pull --rebase origin main
            git push origin main
          else
            echo "No changes to commit"
          fi

## 📚 Content Categories

This repository covers four main DevOps areas with regularly updated content:

### 1. DevOps Interview Questions
- Comprehensive interview questions and answers
- Real-world scenarios and best practices
- [Browse All Interview Content](./devops-interview/)

### 2. Docker
- Containerization techniques
- Docker best practices
- Optimization strategies
- [Browse All Docker Content](./docker/)

### 3. Kubernetes
- Orchestration concepts
- Cluster management
- Advanced deployment strategies
- [Browse All Kubernetes Content](./kubernetes/)

### 4. CI/CD
- Continuous Integration/Continuous Deployment
- Pipeline optimization
- Automation strategies
- [Browse All CI/CD Content](./cicd/)

## 📊 Latest Activity

Check out our [Dashboard](./DASHBOARD.md) for the most recently generated content across all categories.

## 🔄 How It Works

This repository is powered by automated GitHub Actions that:
- Generate fresh content daily using AI
- Update statistics and badges automatically
- Maintain organized content by category
- Provide consistent formatting and quality

## 🤖 Auto-Generated Content

New content is generated daily at 00:00 UTC with random topics covering various DevOps domains. Each piece of content includes:
- Relevant DevOps topics
- Different writing styles (Enterprise, SRE-focused, Security-heavy, etc.)
- Varying detail levels (Low, Medium, High)
- Unique keywords for easy tracking

## 📈 Weekly Summary

For a comprehensive overview of weekly content, check our [Weekly Summary](./WEEKLY_SUMMARY.md).

---

*This repository is automatically maintained. New content is added daily to help you stay current with DevOps trends and best practices.*