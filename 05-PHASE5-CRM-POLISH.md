# Phase 5: CRM, Analytics & Production Polish (Weeks 14-16)

This phase focuses on empowering salespeople with robust CRM tools, analytics for performance tracking, and finalizing the platform for production.

## 5.1 Sales CRM & Pipeline

### Mongoose Schemas

```javascript
const mongoose = require('mongoose');

const LeadSchema = new mongoose.Schema({
  company_id: { type: mongoose.Schema.Types.ObjectId, ref: 'Company', required: true },
  prospect_company_id: { type: mongoose.Schema.Types.ObjectId, ref: 'Company' },
  prospect_contact: { 
    name: String, email: String, phone: String, role: String 
  },
  source: { type: String, enum: ['map_discovery', 'feed', 'ai_recommendation', 'referral', 'import', 'manual'] },
  owner_id: { type: mongoose.Schema.Types.ObjectId, ref: 'User' },
  product_interest: [{ type: mongoose.Schema.Types.ObjectId, ref: 'Product' }],
  qualification: {
    budget: { value: Number, notes: String },
    authority: { hasAuthority: Boolean, notes: String },
    need: { notes: String },
    timeline: { date: Date, notes: String }
  },
  stage: { 
    type: String, 
    enum: ['new', 'qualified', 'contacted', 'meeting', 'proposal', 'negotiation', 'won', 'lost', 'nurture'],
    default: 'new'
  },
  estimated_value: Number,
  expected_close_date: Date,
  loss_reason: String,
  tags: [String],
  activity_count: { type: Number, default: 0 },
  last_activity_at: Date,
  next_followup_at: Date
}, { timestamps: true });

const OpportunitySchema = new mongoose.Schema({
  lead_id: { type: mongoose.Schema.Types.ObjectId, ref: 'Lead', required: true },
  product_id: { type: mongoose.Schema.Types.ObjectId, ref: 'Product' },
  estimated_value: Number,
  stage: String,
  close_date: Date,
  proposal_versions: [{ type: mongoose.Schema.Types.ObjectId, ref: 'Proposal' }],
  negotiation_notes: String,
  outcome: { type: String, enum: ['won', 'lost', 'pending'] },
  won_date: Date,
  lost_date: Date,
  lost_reason: String
}, { timestamps: true });

const ActivitySchema = new mongoose.Schema({
  related_to: { type: mongoose.Schema.Types.ObjectId, refPath: 'onModel' },
  onModel: { type: String, enum: ['Lead', 'Opportunity'] },
  type: { type: String, enum: ['call', 'email', 'meeting', 'note', 'task', 'visit'] },
  owner_id: { type: mongoose.Schema.Types.ObjectId, ref: 'User' },
  title: String,
  description: String,
  due_date: Date,
  completed_at: Date,
  outcome: String
}, { timestamps: true });
```

### API Examples (Express)

```javascript
// GET /api/leads/pipeline
router.get('/pipeline', async (req, res) => {
  const { company_id } = req.user;
  const pipeline = await Lead.aggregate([
    { $match: { company_id: mongoose.Types.ObjectId(company_id) } },
    { $group: {
        _id: "$stage",
        count: { $sum: 1 },
        totalValue: { $sum: "$estimated_value" },
        leads: { $push: "$$ROOT" }
    }}
  ]);
  res.json(pipeline);
});

// PATCH /api/leads/:id/stage
router.patch('/:id/stage', async (req, res) => {
  const { stage } = req.body;
  // Validation logic here...
  const lead = await Lead.findByIdAndUpdate(req.params.id, { stage }, { new: true });
  res.json(lead);
});
```

### React Component Structure
- `PipelineBoard`: Kanban view rendering lists of `PipelineColumn` containing `LeadCard`s. Uses `react-beautiful-dnd` or `@dnd-kit/core`.
- `LeadDetailPage`: Main container. 
  - Tabs: `OverviewTab`, `ActivitiesTab`, `ProposalsTab`, `AIInsightsTab`.
- `ActivityTimeline`: Renders a chronological list of activities.
- `QuickActions`: Floating action bar for quick logging.

## 5.2 Proposal Builder
- **Model**: Tracks opportunity_id, line items, taxes, status, dates.
- **Components**: `ProposalEditor` (form-based builder), `ProposalPreview` (renders HTML/PDF output).

## 5.3 Multi-Meeting Visit Planner
- **Route Optimization**: Integrates with OSRM (Open Source Routing Machine).
- **Components**: `VisitPlannerPage` includes a `MapboxRoute` view and an `ItineraryList`.

## 5.4 Analytics Dashboards
- Uses **Recharts** for visualizations (LineCharts, BarCharts, PieCharts).
- **MongoDB Aggregation Pipeline** handles crunching data for conversions, values by stage, etc.

## 5.5 Platform Admin
- Central hub for `AdminLayout`. Modules: User Mgmt, Content Moderation, Health Monitoring.

## 5.6 Production Polish
- **PWA**: `manifest.json`, `next-pwa` or service worker setup.
- **UX**: Fallbacks, loading skeletons, `react-toastify`.
- **Security**: `helmet`, `express-rate-limit`.

## 5.7 Deployment
- **Docker Compose**: Orchestrates React (Nginx), Express (Node), Python (FastAPI).
- **CI/CD**: GitHub Actions for automated linting, testing, and Docker image publishing.

## 5.8 Week-by-Week Breakdown
- **Week 14**: Lead/Opportunity/Activity models, pipeline board, lead detail pages
- **Week 15**: Proposal builder, visit planner, route optimization, analytics dashboards
- **Week 16**: Admin panel, PWA, responsive audit, security, deployment
