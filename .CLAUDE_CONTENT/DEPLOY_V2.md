# 🚀 V2 Production Deployment Guide

## Overview
This guide covers deploying WuffChat V2 to production by replacing the main branch. V1 is currently in `main`, V2 is in `refactor/prompts`. We deploy via GitHub → Scalingo.

## Pre-Deployment Status ✅
- ✅ V2 architecture complete (clean FSM, separation of concerns)
- ✅ All tests passing (119 tests, 0 failures)
- ✅ Specific error messages implemented
- ✅ Postman collection verified
- ✅ Production-ready code quality

## Deployment Strategy: Direct Main Branch Replacement

Since V1 is no longer needed in production, we'll replace `main` entirely with V2 content. This is the cleanest approach for your setup.

---

## Step 1: Pre-Deployment Verification

### 1.1 Final Code Check
```bash
# Ensure you're on refactor/prompts with clean state
git checkout refactor/prompts
git status  # Should show "working tree clean"

# Run final test suite
pytest -v
# Expected: 119 passed, 61 skipped

# Verify V2 API starts correctly
python -m uvicorn src.v2.main:app --port 8000
# Test: http://localhost:8000/docs should show V2 API
```

### 1.2 Postman Final Verification
- Run complete Postman collection against local V2
- Verify all error scenarios work (short input, no match, invalid yes/no)
- Confirm response format matches expected frontend needs

---

## Step 2: Create V1 Backup

### 2.1 Backup Current Production
```bash
# Switch to main and create timestamped backup
git checkout main
git tag -a v1-backup-$(date +%Y%m%d) -m "V1 backup before V2 deployment"
git push origin v1-backup-$(date +%Y%m%d)

# Verify backup created
git tag | grep v1-backup
```

### 2.2 Document Current State
```bash
# Record current main commit for documentation
git log --oneline -1 main
# Save this commit hash for rollback reference
```

---

## Step 3: Deploy V2 to Main Branch

### 3.1 Replace Main with V2
```bash
# Hard reset main to match refactor/prompts exactly
git reset --hard refactor/prompts
git push origin main --force-with-lease
```

### 3.2 Verify Push Success
```bash
# Confirm main now contains V2
git log --oneline -5 main
# Should show recent V2 commits

# Check GitHub to confirm push
# Visit: https://github.com/kemperfekt/dogbot-agent
# Main branch should now show V2 code structure
```

---

## Step 4: Scalingo Auto-Deployment

### 4.1 Monitor Deployment
- Scalingo will automatically detect the push to `main`
- Monitor deployment in Scalingo dashboard
- Build should complete successfully using existing container setup

### 4.2 Deployment Logs
```bash
# If you have Scalingo CLI installed
scalingo --app your-app-name logs --lines 100

# Watch for:
# - Successful dependency installation
# - V2 API startup messages
# - No error messages during boot
```

---

## Step 5: Production Verification

### 5.1 Health Check
```bash
# Test V2 health endpoint
curl https://api.wuffchat.de/v2/health

# Expected response:
{
  "orchestrator": "healthy",
  "flow_engine": "healthy",
  "services": { ... },
  "overall": "healthy"
}
```

### 5.2 API Functionality Test
```bash
# Test flow_intro endpoint
curl -X POST https://api.wuffchat.de/flow_intro \
  -H "Content-Type: application/json" \
  -d '{"message": ""}'

# Expected: Session ID + greeting messages

# Test flow_step with short input (error message test)
curl -X POST https://api.wuffchat.de/flow_step \
  -H "Content-Type: application/json" \
  -d '{"session_id": "SESSION_ID_FROM_ABOVE", "message": "Hi"}'

# Expected: "Das ist etwas kurz. Kannst du mir mehr Details geben?"
```

### 5.3 Full Postman Test
- Run Postman collection against production URLs
- Update environment to use `api.wuffchat.de`
- Verify all test scenarios pass

---

## Step 6: Frontend Update (Separate Repository)

### 6.1 Frontend Changes Needed
In `dogbot-ui` repository:

```javascript
// Update API endpoints (if not already using these)
const API_BASE = 'https://api.wuffchat.de';

// API calls should already work since V2 maintains compatibility
// POST /flow_intro
// POST /flow_step

// Response format is identical to V1, so minimal changes needed
```

### 6.2 Frontend Testing
1. Test against new V2 backend
2. Verify conversation flow works end-to-end
3. Test error scenarios (short input, invalid responses)
4. Deploy frontend changes

---

## Rollback Plan 🔄

### If Issues Arise
```bash
# Quick rollback to V1
git checkout main
git reset --hard v1-backup-$(date +%Y%m%d)
git push origin main --force-with-lease

# Scalingo will auto-deploy V1 again
```

### Rollback Verification
```bash
# Verify V1 is back
curl https://api.wuffchat.de/docs
# Should show V1 API documentation

# Test basic functionality
curl -X POST https://api.wuffchat.de/flow_intro \
  -H "Content-Type: application/json" \
  -d '{"message": ""}'
```

---

## Post-Deployment Monitoring

### 📊 Metrics to Watch
- **Response times**: Should be ≤ V1 performance
- **Error rates**: Should be < 1%
- **User flow completion**: Monitor for any new drop-off points
- **Memory usage**: V2 should be more efficient

### 🔍 Logs to Monitor
```bash
# Key log patterns to watch for
scalingo --app your-app-name logs --lines 1000 | grep -i error
scalingo --app your-app-name logs --lines 1000 | grep -i "v2"

# Good signs:
# - "V2 handling message for session..."
# - "Generated X response messages"
# - No Python tracebacks
```

### 📱 User Experience
- Test complete user journeys
- Verify error messages are helpful
- Confirm conversation flow feels natural

---

## Success Criteria ✅

### Deployment Successful When:
- [ ] V2 health endpoint returns "healthy"
- [ ] Postman collection passes 100% on production
- [ ] Frontend can complete full conversation flow
- [ ] Error messages are specific and helpful
- [ ] No Python errors in logs
- [ ] Response times ≤ V1 baseline

### Rollback Required If:
- ❌ Health endpoint fails
- ❌ Critical errors in logs
- ❌ Frontend cannot connect
- ❌ User flow broken
- ❌ Response times significantly worse

---

## Why This Approach Works

### ✅ **Advantages**
- **Simple**: One deployment replaces everything
- **Clean**: No parallel systems or feature flags
- **Safe**: V1 backup available for instant rollback
- **Fast**: Immediate V2 benefits for all users
- **Low Risk**: Comprehensive test coverage + same API surface

### 🔧 **Technical Benefits**
- Uses existing Scalingo infrastructure
- No container changes needed
- Same environment variables work
- Database connections unchanged

---

## Emergency Contacts & Resources

### 🆘 **If Something Goes Wrong**
1. **Check logs first**: `scalingo logs`
2. **Rollback immediately**: Use rollback commands above
3. **Verify rollback**: Test V1 endpoints work
4. **Debug later**: Investigate issues in development

### 📚 **References**
- **V2 Health Check**: `GET /v2/health`
- **API Documentation**: `https://api.wuffchat.de/docs`
- **Test Collection**: `tests/postman/postman_collection.json`
- **Backup Tags**: `git tag | grep v1-backup`

---

**Ready to deploy? Your V2 is excellent and production-ready! 🚀**

*Last updated: May 30, 2025*