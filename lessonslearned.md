# Lessons Learned

A running log of issues we encountered and how we fixed them.

---

## 2026-06-03 - Pokemon Card Price Lookup Tool

### Issue 1: Search Not Working (0 Cards Found)
**Problem:** Searching for cards like "Pikachu" returned 0 results, but "Charizard" worked.

**Cause:** The search query was using exact match with quotes: `name:"pikachu"` which required exact capitalization.

**Solution:** Removed quotes from search query to use fuzzy matching: `name:pikachu` instead. This makes it case-insensitive.

**Files Changed:** `/var/www/retroblasts.com/pokemon-cards.html` (lines 438, 440)

---

### Issue 2: All Prices Showing N/A
**Problem:** Every card showed "N/A" for all prices (raw/ungraded and PSA grades).

**Cause:**
1. PriceCharting web scraping wasn't returning data
2. Fallback API wasn't working properly
3. No fallback to TCGPlayer data already in the Pokemon TCG API response

**Solution:** Updated `fetchRealPrices()` function to:
1. First check if card has TCGPlayer pricing data (already included in Pokemon TCG API)
2. Use that as base price and calculate estimated PSA grades (1.35x, 1.75x, 2.5x, 4x)
3. Try to fetch real PSA prices in background (non-blocking)
4. Update display if real prices become available

**Files Changed:** `/var/www/retroblasts.com/pokemon-cards.html` (lines 593-702)

---

### Issue 3: Page Not Loading / JavaScript Breaking
**Problem:** After fixing prices, searching caused page to reload but nothing appeared. Coin balance stuck on "Loading..."

**Cause:** Extra closing brace `}` in JavaScript causing syntax error that broke all JavaScript on the page.

**Solution:** Removed duplicate closing brace at line 703.

**Files Changed:** `/var/www/retroblasts.com/pokemon-cards.html` (line 703 deleted)

---

## 2026-06-01 - MDC Assembly Service Area Map

### Issue: Merging Git Conflicts
**Problem:** Server had 6 local commits with Dad's accurate NY state map, but different styling than what was pushed to GitHub.

**Solution:**
1. Aborted initial merge with conflicts
2. Made changes directly on server to combine Dad's accurate geography with red markers
3. Committed server version and merged with GitHub
4. Used `git checkout --ours` to keep server version during conflicts
5. Pulled changes back to local repo

**Lesson:** When server and local have diverged significantly, sometimes it's easier to make the final changes directly where they're needed and merge back.

---

## Template for Future Entries

### Issue: [Brief Description]
**Problem:** [What went wrong]

**Cause:** [Why it happened]

**Solution:** [How we fixed it]

**Files Changed:** [List of files modified]

**Lesson:** [What we learned for next time]

---

**Last Updated:** 2026-06-03

---

## 2026-06-03 - Website Audit Tool API Failure

### Issue: All Website Audits Failing
**Problem:** Every website URL entered into the audit tool returned "Error: Failed to analyze website. Please check the URL and try again."

**Cause:** The audit API was trying to use Google PageSpeed Insights API, which:
1. Requires an API key (we did not have one)
2. Has strict rate limits
3. Can be slow (30+ seconds)
4. Sometimes times out

**Solution:** Rewrote the audit API to do its own analysis:
- Fetches the page directly with curl
- Measures actual load time
- Parses HTML with DOMDocument to check meta description, title tag, image alt tags, mobile viewport tag
- Checks SSL (https)
- Calculates scores based on findings
- No external API dependencies
- No rate limits
- Fast results (1-2 seconds)

**Files Changed:** /var/www/freshbuild.co/api/audit.php

**Lesson:** When building tools that depend on external APIs, always have a fallback or build your own analysis. External APIs can have rate limits, API key requirements, downtime, and slow response times. Building our own analysis gives us full control and reliability.

---

**Last Updated:** 2026-06-03
