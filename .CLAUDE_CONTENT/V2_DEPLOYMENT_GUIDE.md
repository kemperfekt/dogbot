# 🚀 WuffChat V2 Deployment Process

## Overview
This guide walks through the complete process of deploying your excellent V2 to production by replacing V1 in the main branch.

## Current Situation Assessment ✅
- ✅ **V1**: Currently in `main` branch (production)
- ✅ **V2**: Complete and tested in `refactor/prompts` branch
- ✅ **Test Status**: 119 tests passing, 0 failures
- ✅ **Code Quality**: Production-ready

---

## 📋 Pre-Deployment Checklist

### Step 1: Verify V2 Readiness
```bash
# Ensure you're on refactor/prompts with clean state
git checkout refactor/prompts
git status
# Should show: "On branch refactor/prompts, nothing to commit, working tree clean"

# Final test run
pytest -v
# Expected: 119 passed, 61 skipped

# Verify V2 API starts correctly
python -m uvicorn src.v2.main:app --port 8000
# Test: http://localhost:8000/docs should show V2 API docs
```

### Step 2: Document Current State
```bash
# Record current commit hashes for reference
echo "V1 (main) commit:" > deployment-log.txt
git log --oneline -1 main >> deployment-log.txt

echo "V2 (refactor/prompts) commit:" >> deployment-log.txt
git log --oneline -1 refactor/prompts >> deployment-log.txt

echo "Deployment date: $(date)" >> deployment-log.txt
```

---

## 🏷️ Step-by-Step Deployment Process

### Step 3: Create V1 Backup Tag
```bash
# Switch to main and create backup tag
git checkout main
git tag -a v1-backup-$(date +%Y%m%d) -m "V1 backup before V2 deployment - $(date)"
git push origin v1-backup-$(date +%Y%m%d)

# Verify tag was created
git tag | grep v1-backup
echo "V1 backup tag created: v1-backup-$(date +%Y%m%d)" >> deployment-log.txt
```

### Step 4: Compare V1 vs V2 (Understanding the Changes)
```bash
# Compare branch structures
echo "=== V1 vs V2 Comparison ===" >> deployment-log.txt
echo "Files only in V1:" >> deployment-log.txt
git diff --name-only main refactor/prompts | grep "^-" >> deployment-log.txt

echo "Files only in V2:" >> deployment-log.txt  
git diff --name-only main refactor/prompts | grep "^+" >> deployment-log.txt

echo "Modified files:" >> deployment-log.txt
git diff --name-only main refactor/prompts | grep -v "^[+-]" >> deployment-log.txt

# Show key differences
echo "=== Key Architecture Changes ===" >> deployment-log.txt
echo "- V1: src/flow/, src/agents/" >> deployment-log.txt
echo "- V2: src/v2/ (complete rewrite)" >> deployment-log.txt
echo "- API: Same endpoints (/flow_intro, /flow_step)" >> deployment-log.txt
echo "- Tests: 119 passing vs V1's limited tests" >> deployment-log.txt
```

### Step 5: Deploy V2 to Main Branch
```bash
# Replace main with V2 content (the moment of truth!)
git reset --hard refactor/prompts

# Force push to main (with safety check)
git push origin main --force-with-lease

# Verify the push succeeded
git log --oneline -5 main
echo "V2 deployed to main at: $(date)" >> deployment-log.txt
```

### Step 6: Verify Deployment Success
```bash
# Confirm main now contains V2
echo "=== Post-Deployment Verification ===" >> deployment-log.txt
echo "Main branch now contains:" >> deployment-log.txt
ls -la src/ >> deployment-log.txt

# Check key V2 files exist
if [ -f "src/v2/main.py" ]; then
    echo "✅ V2 main.py present" >> deployment-log.txt
else
    echo "❌ V2 main.py missing!" >> deployment-log.txt
fi

if [ -f "src/v2/core/flow_engine.py" ]; then
    echo "✅ V2 flow_engine.py present" >> deployment-log.txt
else
    echo "❌ V2 flow_engine.py missing!" >> deployment-log.txt
fi

# Verify tests still pass on main
git checkout main
pytest -v > test-results-main.txt 2>&1
if [ $? -eq 0 ]; then
    echo "✅ Tests passing on main branch" >> deployment-log.txt
else
    echo "❌ Tests failing on main branch!" >> deployment-log.txt
fi
```

---

## 🔍 V1 vs V2 Architecture Comparison

### Major Changes Overview
```bash
# Generate detailed comparison report
echo "=== DETAILED V1 vs V2 COMPARISON ===" > v1-vs-v2-comparison.md

echo "## File Structure Changes" >> v1-vs-v2-comparison.md
echo "### Removed (V1 only):" >> v1-vs-v2-comparison.md
echo "- src/flow/flow_orchestrator.py (500+ lines → replaced by V2 FSM)" >> v1-vs-v2-comparison.md
echo "- src/agents/ (old agent implementations)" >> v1-vs-v2-comparison.md
echo "- src/services/ (V1 service implementations)" >> v1-vs-v2-comparison.md

echo "### Added (V2 only):" >> v1-vs-v2-comparison.md
echo "- src/v2/ (complete new architecture)" >> v1-vs-v2-comparison.md
echo "- src/v2/core/flow_engine.py (FSM implementation)" >> v1-vs-v2-comparison.md
echo "- src/v2/core/prompt_manager.py (centralized prompts)" >> v1-vs-v2-comparison.md
echo "- tests/v2/ (119 comprehensive tests)" >> v1-vs-v2-comparison.md

echo "### Maintained:" >> v1-vs-v2-comparison.md
echo "- API endpoints (/flow_intro, /flow_step)" >> v1-vs-v2-comparison.md
echo "- Response format (frontend compatibility)" >> v1-vs-v2-comparison.md
echo "- Environment variables" >> v1-vs-v2-comparison.md
```

### API Compatibility Check
```bash
# Test API compatibility
echo "## API Compatibility Test" >> v1-vs-v2-comparison.md

# Start V2 API in background for testing
python -m uvicorn src.v2.main:app --port 8001 &
API_PID=$!
sleep 5

# Test key endpoints
echo "Testing V2 endpoints:" >> v1-vs-v2-comparison.md

# Test flow_intro
INTRO_RESPONSE=$(curl -s -X POST http://localhost:8001/flow_intro \
  -H "Content-Type: application/json" \
  -d '{"message": ""}')

if echo "$INTRO_RESPONSE" | grep -q "session_id"; then
    echo "✅ /flow_intro working" >> v1-vs-v2-comparison.md
else
    echo "❌ /flow_intro failing" >> v1-vs-v2-comparison.md
fi

# Test health endpoint
HEALTH_RESPONSE=$(curl -s http://localhost:8001/v2/health)
if echo "$HEALTH_RESPONSE" | grep -q "healthy"; then
    echo "✅ /v2/health working" >> v1-vs-v2-comparison.md
else
    echo "❌ /v2/health failing" >> v1-vs-v2-comparison.md
fi

# Stop test API
kill $API_PID 2>/dev/null

echo "## Summary" >> v1-vs-v2-comparison.md
echo "V2 maintains full API compatibility while providing:" >> v1-vs-v2-comparison.md
echo "- ✅ Better architecture (FSM vs hardcoded logic)" >> v1-vs-v2-comparison.md
echo "- ✅ Comprehensive tests (119 vs minimal)" >> v1-vs-v2-comparison.md
echo "- ✅ Specific error messages" >> v1-vs-v2-comparison.md
echo "- ✅ Centralized prompts" >> v1-vs-v2-comparison.md
echo "- ✅ Better maintainability" >> v1-vs-v2-comparison.md
```

---

## 🚀 GitHub Actions Setup for Future Deployments

### Create GitHub Actions Workflow
```yaml
# .github/workflows/deploy.yml
name: WuffChat V2 Deployment

on:
  push:
    branches: [main]
    paths: ['src/**', 'requirements.txt', 'tests/**']
  pull_request:
    branches: [main]

env:
  PYTHON_VERSION: '3.11'

jobs:
  test:
    runs-on: ubuntu-latest
    name: Test Suite
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: ${{ env.PYTHON_VERSION }}
          
      - name: Cache dependencies
        uses: actions/cache@v3
        with:
          path: ~/.cache/pip
          key: ${{ runner.os }}-pip-${{ hashFiles('requirements.txt') }}
          
      - name: Install dependencies
        run: |
          pip install -r requirements.txt
          pip install -r requirements-dev.txt
          
      - name: Run tests
        run: |
          pytest -v --cov=src --cov-report=xml
          
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        
  api-test:
    runs-on: ubuntu-latest
    name: API Integration Test
    needs: test
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: ${{ env.PYTHON_VERSION }}
          
      - name: Install dependencies
        run: |
          pip install -r requirements.txt
          
      - name: Start API server
        run: |
          python -m uvicorn src.v2.main:app --port 8000 &
          sleep 10
          
      - name: Test API endpoints
        run: |
          # Test health endpoint
          curl -f http://localhost:8000/v2/health
          
          # Test flow_intro
          RESPONSE=$(curl -s -X POST http://localhost:8000/flow_intro \
            -H "Content-Type: application/json" \
            -d '{"message": ""}')
          echo "$RESPONSE" | grep -q "session_id"
          
          # Test error handling
          ERROR_RESPONSE=$(curl -s -X POST http://localhost:8000/flow_step \
            -H "Content-Type: application/json" \
            -d '{"session_id": "test", "message": "Hi"}')
          echo "$ERROR_RESPONSE" | grep -q "Das ist etwas kurz"

  deploy-staging:
    runs-on: ubuntu-latest
    name: Deploy to Staging
    needs: [test, api-test]
    if: github.ref == 'refs/heads/main'
    environment: staging
    
    steps:
      - name: Deploy to Staging
        run: |
          echo "Deploying to staging environment..."
          # Add your staging deployment commands here
          # e.g., git push staging main
          
  deploy-production:
    runs-on: ubuntu-latest
    name: Deploy to Production
    needs: deploy-staging
    if: github.ref == 'refs/heads/main'
    environment: production
    
    steps:
      - name: Deploy to Production
        run: |
          echo "Deploying to production environment..."
          # Add your production deployment commands here
          # e.g., git push production main
```

### Create Environment Protection Rules
```bash
# Create GitHub environments (do this in GitHub UI)
echo "TODO: Create GitHub environments:"
echo "1. Go to Settings > Environments in your GitHub repo"
echo "2. Create 'staging' environment"
echo "3. Create 'production' environment"
echo "4. Add protection rules for production (require approval)"
```

---

## 🔄 Rollback Plan (If Needed)

### Emergency Rollback to V1
```bash
# If something goes wrong, rollback immediately
git checkout main
git reset --hard v1-backup-$(date +%Y%m%d)
git push origin main --force-with-lease

echo "Rolled back to V1 at: $(date)" >> deployment-log.txt
echo "Reason: [ADD REASON HERE]" >> deployment-log.txt
```

### Rollback Verification
```bash
# Verify V1 is working after rollback
python -m uvicorn src.main:app --port 8000 &
sleep 5

# Test V1 endpoint
curl -f http://localhost:8000/docs
if [ $? -eq 0 ]; then
    echo "✅ V1 rollback successful" >> deployment-log.txt
else
    echo "❌ V1 rollback failed!" >> deployment-log.txt
fi

# Stop test server
pkill -f uvicorn
```

---

## 📊 Post-Deployment Verification Checklist

### Immediate Checks (5 minutes)
```bash
# 1. API Health
curl https://your-domain.com/v2/health
# Expected: {"overall": "healthy", ...}

# 2. Basic Flow Test
curl -X POST https://your-domain.com/flow_intro \
  -H "Content-Type: application/json" \
  -d '{"message": ""}'
# Expected: session_id + greeting messages

# 3. Error Message Test
curl -X POST https://your-domain.com/flow_step \
  -H "Content-Type: application/json" \
  -d '{"session_id": "test", "message": "Hi"}'
# Expected: "Das ist etwas kurz. Kannst du mir mehr Details geben?"
```

### Extended Verification (30 minutes)
```bash
# 1. Run full Postman collection against production
# 2. Test frontend integration
# 3. Monitor logs for errors
# 4. Check performance metrics
# 5. Verify all conversation flows work
```

---

## 🎯 Success Criteria

### Deployment Successful When:
- [ ] ✅ V2 health endpoint returns "healthy"
- [ ] ✅ All API endpoints respond correctly
- [ ] ✅ Error messages are specific (not generic)
- [ ] ✅ Frontend can complete conversation flows
- [ ] ✅ No Python errors in logs
- [ ] ✅ Response times ≤ baseline
- [ ] ✅ GitHub Actions pipeline is green

### Additional Monitoring:
- [ ] Check server logs for any errors
- [ ] Monitor response times
- [ ] Verify conversation completion rates
- [ ] Test multiple user scenarios

---

## 📝 Deployment Log Template

```bash
# Create final deployment report
cat > DEPLOYMENT_REPORT.md << EOF
# V2 Deployment Report

**Date**: $(date)
**Branch**: main ← refactor/prompts
**Deployer**: $(git config user.name)

## Pre-Deployment Status
- V1 Backup Tag: v1-backup-$(date +%Y%m%d)
- V2 Tests: 119 passing
- Code Quality: Production-ready

## Deployment Actions
- [x] Created V1 backup tag
- [x] Merged V2 to main
- [x] Verified API functionality
- [x] Set up GitHub Actions

## Post-Deployment Verification
- API Health: ✅/❌
- Basic Flow: ✅/❌
- Error Messages: ✅/❌
- Performance: ✅/❌

## Issues Encountered
[None/List any issues]

## Next Steps
- [ ] Monitor production logs
- [ ] Test frontend integration
- [ ] Update documentation
- [ ] Plan next features (profiles!)

**Status**: SUCCESS/ROLLBACK REQUIRED
EOF
```

---

## 🚀 Ready to Execute?

### Quick Checklist Before Starting:
1. **Clean working directory**: `git status` shows clean
2. **On correct branch**: `git branch` shows `refactor/prompts`
3. **Tests passing**: `pytest -v` shows 119 passed
4. **Backup strategy clear**: Know how to rollback if needed
5. **Monitoring ready**: Can check logs/health after deployment

### The deployment process is:
1. **5 minutes**: Create backup tag
2. **2 minutes**: Merge to main
3. **10 minutes**: Verify deployment
4. **15 minutes**: Set up GitHub Actions

**Total time**: ~30 minutes to have V2 in production with CI/CD! 🎉

Your V2 is excellent and ready - this deployment will be a major upgrade for WuffChat!

**Ready to start? Let me know and I'll guide you through each step!** 🚀