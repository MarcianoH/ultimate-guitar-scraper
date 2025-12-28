# Product Requirements Document (PRD)
## Ultimate Guitar Scraper

**Version:** 1.0.0 | **Date:** December 27, 2025 | **Status:** In Development

---

## 1. Executive Summary

The Ultimate Guitar Scraper is a headless, serverless web scraper that automatically downloads guitar tabs from Ultimate Guitar without login. Key features:

- ✅ **No Login Required** - Scrapes public content
- - ✅ **Resumable Checkpoints** - Never loses progress
  - - ✅ **Auto-Recovery** - Restarts automatically on failure
    - - ✅ **Background Operation** - Runs 24/7 continuously
      - - ✅ **REST API** - Query downloaded tabs programmatically
        - - ✅ **Offline-First** - Access content without internet
         
          - ---

          ## 2. Problem Statement

          Ultimate Guitar hosts millions of tabs but users face:
          - Slow browsing due to site load times
          - - No bulk download capabilities
            - - Dependency on internet connectivity
              - - Missing offline access to favorite songs
                - - Inefficient tab discovery
                 
                  - **Solution:** Automatically discover, download, and index all tabs into a local SQLite database with REST API access.
                 
                  - ---

                  ## 3. Target Users

                  1. **Music Teachers** - Need bulk access for lesson planning
                  2. 2. **Guitar Enthusiasts** - Want personal archives
                     3. 3. **Developers** - Building music education platforms
                        4. 4. **Music Curators** - Creating comprehensive databases
                          
                           5. ---
                          
                           6. ## 4. Core Features (MVP)
                          
                           7. ### Feature 1: Headless Tab Scraping
                           8. - Use Puppeteer for JavaScript rendering
                              - - Extract metadata: title, artist, difficulty, rating, views
                                - - Parse tab content (chords/tabs text)
                                  - - Handle dynamic content loading
                                    - - User-Agent rotation
                                      - - Rate limiting (3-5 second delays)
                                       
                                        - **Acceptance Criteria:**
                                        - - Successfully scrape 100% of public tabs
                                          - - Extract all metadata fields
                                            - - Handle JavaScript-rendered content
                                              - - No rate-limiting triggers
                                               
                                                - ### Feature 2: Checkpointed Progress
                                                - - SQLite database for persistent state
                                                  - - Save progress every 25 tabs
                                                    - - Transactional support for crash safety
                                                      - - Hourly automated backups
                                                       
                                                        - **Acceptance Criteria:**
                                                        - - Zero data loss on unexpected shutdown
                                                          - - Automatic recovery of pending items
                                                            - - Database integrity verified
                                                             
                                                              - ### Feature 3: Auto-Recovery & Restart
                                                              - - PM2 process manager
                                                                - - Max 10 restarts per hour
                                                                  - - 60-second restart delay
                                                                    - - Health check on database connectivity
                                                                     
                                                                      - **Acceptance Criteria:**
                                                                      - - Restart within 60 seconds of crash
                                                                        - - No manual intervention required
                                                                          - - All recovery events logged
                                                                           
                                                                            - ### Feature 4: URL Discovery & Queue
                                                                            - - Extract related tabs from pages
                                                                              - - Discover via artist pages
                                                                                - - Queue prioritization
                                                                                  - - Prevent duplicates
                                                                                   
                                                                                    - **Acceptance Criteria:**
                                                                                    - - Discover all related tabs
                                                                                      - - No duplicate processing
                                                                                        - - 50,000+ unique URLs discoverable
                                                                                         
                                                                                          - ### Feature 5: Error Handling
                                                                                          - - Retry with exponential backoff (1s, 2s, 5s, 10s)
                                                                                            - - Max 3 attempts per URL
                                                                                              - - Comprehensive error logging
                                                                                                - - 30-second timeout handling
                                                                                                 
                                                                                                  - **Acceptance Criteria:**
                                                                                                  - - All errors logged with context
                                                                                                    - - Retry logic verified in logs
                                                                                                      - - Zero unhandled crashes
                                                                                                       
                                                                                                        - ### Feature 6: REST API
                                                                                                        - - Search tabs by title/artist
                                                                                                          - - Fetch single tab by ID
                                                                                                            - - List all tabs with pagination
                                                                                                              - - Get statistics and queue length
                                                                                                                - - Health check endpoint
                                                                                                                 
                                                                                                                  - **Acceptance Criteria:**
                                                                                                                  - - All endpoints return JSON
                                                                                                                    - - Search <200ms response
                                                                                                                      - - Pagination working
                                                                                                                        - - 24/7 availability
                                                                                                                         
                                                                                                                          - ---
                                                                                                                          
                                                                                                                          ## 5. Technical Specifications
                                                                                                                          
                                                                                                                          ### Stack
                                                                                                                          - **Runtime:** Node.js 18+
                                                                                                                          - - **Browser:** Puppeteer (headless)
                                                                                                                            - - **Database:** SQLite 3
                                                                                                                              - - **API:** Express.js
                                                                                                                                - - **Process Manager:** PM2
                                                                                                                                  - - **Logging:** Pino
                                                                                                                                   
                                                                                                                                    - ### Database Schema
                                                                                                                                   
                                                                                                                                    - **tabs table:**
                                                                                                                                    - ```
                                                                                                                                      - id, url (UNIQUE), tab_id, artist_slug, song_slug
                                                                                                                                      - title, artist, difficulty, genre, rating, views, saves
                                                                                                                                      - author, content, content_hash (UNIQUE)
                                                                                                                                      - tuning, capo, key, created_at, updated_at
                                                                                                                                      - status (pending/in_progress/completed/failed)
                                                                                                                                      - error_message, attempt_count
                                                                                                                                      ```
                                                                                                                                      
                                                                                                                                      **queue table:**
                                                                                                                                      ```
                                                                                                                                      - id, url (UNIQUE), priority, retry_count, added_at
                                                                                                                                      ```
                                                                                                                                      
                                                                                                                                      **discovered_urls table:**
                                                                                                                                      ```
                                                                                                                                      - id, url (UNIQUE), source_url, discovered_at
                                                                                                                                      ```
                                                                                                                                      
                                                                                                                                      ---
                                                                                                                                      
                                                                                                                                      ## 6. REST API Endpoints
                                                                                                                                      
                                                                                                                                      ### Search Tabs
                                                                                                                                      ```
                                                                                                                                      GET /api/search?q=query
                                                                                                                                      Response: Array of matching tabs
                                                                                                                                      ```
                                                                                                                                      
                                                                                                                                      ### Get Tab by ID
                                                                                                                                      ```
                                                                                                                                      GET /api/tab/:id
                                                                                                                                      Response: Complete tab object with content
                                                                                                                                      ```
                                                                                                                                      
                                                                                                                                      ### List Tabs (Paginated)
                                                                                                                                      ```
                                                                                                                                      GET /api/tabs?page=1&limit=50
                                                                                                                                      Response: Paginated tab results
                                                                                                                                      ```
                                                                                                                                      
                                                                                                                                      ### Statistics
                                                                                                                                      ```
                                                                                                                                      GET /api/stats
                                                                                                                                      Response: { total, completed, failed, pending }
                                                                                                                                      ```
                                                                                                                                      
                                                                                                                                      ### Queue Status
                                                                                                                                      ```
                                                                                                                                      GET /api/queue
                                                                                                                                      Response: { queue_length, avg_items_per_hour }
                                                                                                                                      ```
                                                                                                                                      
                                                                                                                                      ### Health Check
                                                                                                                                      ```
                                                                                                                                      GET /health
                                                                                                                                      Response: { status, database, scraper_running }
                                                                                                                                      ```
                                                                                                                                      
                                                                                                                                      ---
                                                                                                                                      
                                                                                                                                      ## 7. Performance Targets
                                                                                                                                      
                                                                                                                                      | Metric | Target |
                                                                                                                                      |--------|--------|
                                                                                                                                      | Tabs per hour | 400-500 |
                                                                                                                                      | API response | <200ms |
                                                                                                                                      | Memory usage | <500MB |
                                                                                                                                      | Disk space | ~2GB for 100k tabs |
                                                                                                                                      | Uptime | 99%+ |
                                                                                                                                      
                                                                                                                                      ---
                                                                                                                                      
                                                                                                                                      ## 8. Success Criteria
                                                                                                                                      
                                                                                                                                      **Phase 1 (Weeks 1-2):**
                                                                                                                                      - [ ] Core scraper functional
                                                                                                                                      - [ ] - [ ] SQLite checkpoint system
                                                                                                                                      - [ ] - [ ] Error handling with retries
                                                                                                                                      - [ ] - [ ] Basic logging
                                                                                                                                     
                                                                                                                                      - [ ] **Phase 2 (Weeks 3-4):**
                                                                                                                                      - [ ] - [ ] URL crawler implemented
                                                                                                                                      - [ ] - [ ] Queue management
                                                                                                                                      - [ ] - [ ] PM2 integration
                                                                                                                                      - [ ] - [ ] 10,000+ tabs downloaded
                                                                                                                                     
                                                                                                                                      - [ ] **Phase 3 (Weeks 5-6):**
                                                                                                                                      - [ ] - [ ] REST API working
                                                                                                                                      - [ ] - [ ] Database backups
                                                                                                                                      - [ ] - [ ] Incremental updates
                                                                                                                                      - [ ] - [ ] 50,000+ tabs
                                                                                                                                     
                                                                                                                                      - [ ] **Phase 4 (Weeks 7-8):**
                                                                                                                                      - [ ] - [ ] Performance optimized
                                                                                                                                      - [ ] - [ ] Full-text search
                                                                                                                                      - [ ] - [ ] 100,000+ tabs
                                                                                                                                      - [ ] - [ ] Production ready
                                                                                                                                     
                                                                                                                                      - [ ] ---
                                                                                                                                     
                                                                                                                                      - [ ] ## 9. Deployment
                                                                                                                                     
                                                                                                                                      - [ ] ### Installation
                                                                                                                                      - [ ] ```bash
                                                                                                                                      - [ ] git clone https://github.com/MarcianoH/ultimate-guitar-scraper.git
                                                                                                                                      - [ ] cd ultimate-guitar-scraper
                                                                                                                                      - [ ] npm install
                                                                                                                                      - [ ] mkdir -p data/backups logs
                                                                                                                                      - [ ] cp .env.example .env
                                                                                                                                      - [ ] npm run pm2:start
                                                                                                                                      - [ ] ```
                                                                                                                                     
                                                                                                                                      - [ ] ### Monitoring
                                                                                                                                      - [ ] ```bash
                                                                                                                                      - [ ] pm2 status          # Check status
                                                                                                                                      - [ ] pm2 logs scraper    # View logs
                                                                                                                                      - [ ] pm2 monit           # Monitor resources
                                                                                                                                      - [ ] pm2 restart scraper # Restart if needed
                                                                                                                                      - [ ] ```
                                                                                                                                     
                                                                                                                                      - [ ] ---
                                                                                                                                     
                                                                                                                                      - [ ] ## 10. Risks & Mitigation
                                                                                                                                     
                                                                                                                                      - [ ] | Risk | Mitigation |
                                                                                                                                      - [ ] |------|-----------|
                                                                                                                                      - [ ] | Site blocks scraper | Rate limiting, User-Agent rotation |
                                                                                                                                      - [ ] | Database corruption | Hourly backups, transactions |
                                                                                                                                      - [ ] | Crashes/data loss | Checkpoint system, PM2 auto-restart |
                                                                                                                                      - [ ] | Performance issues | Database indexing, optimization |
                                                                                                                                      - [ ] | ToS violation | Request permission, check robots.txt |
                                                                                                                                     
                                                                                                                                      - [ ] ---
                                                                                                                                     
                                                                                                                                      - [ ] ## 11. Future Features (Phase 2+)
                                                                                                                                     
                                                                                                                                      - [ ] - Full-text search with Elasticsearch
                                                                                                                                      - [ ] - Web dashboard for monitoring
                                                                                                                                      - [ ] - Real-time progress tracking
                                                                                                                                      - [ ] - Incremental updates
                                                                                                                                      - [ ] - Backup & restore UI
                                                                                                                                      - [ ] - Advanced filtering
                                                                                                                                      - [ ] - User ratings
                                                                                                                                      - [ ] - Genre discovery
                                                                                                                                     
                                                                                                                                      - [ ] ---
                                                                                                                                     
                                                                                                                                      - [ ] ## 12. Next Steps
                                                                                                                                     
                                                                                                                                      - [ ] 1. Clone repository locally
                                                                                                                                      - [ ] 2. Install dependencies
                                                                                                                                      - [ ] 3. Run initial scraper test
                                                                                                                                      - [ ] 4. Set up PM2 process manager
                                                                                                                                      - [ ] 5. Begin continuous scraping
                                                                                                                                      - [ ] 6. Monitor progress via logs
                                                                                                                                     
                                                                                                                                      - [ ] **Target:** 10,000 tabs by end of Week 2, 100,000 by end of Month 1
