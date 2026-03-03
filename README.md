# The Agentic Ecom Playbook — Skills Library

Ready-to-use Claude Code skills for e-commerce media buyers. Each skill is a self-contained SKILL.md file that gives Claude Code domain-specific knowledge and step-by-step instructions for automating common e-commerce and performance marketing tasks.

## Installation

Download any skill file and add it to your project's `.claude/skills/` directory:

```bash
# Example: install the catalog-image-gen skill
mkdir -p .claude/skills
cp skills/catalog-image-gen.SKILL.md .claude/skills/
```

Claude Code will automatically discover and use skills placed in `.claude/skills/`.

## Available Skills

| # | Skill | Description |
|---|-------|-------------|
| 1 | [catalog-image-gen](skills/catalog-image-gen.SKILL.md) | Generate catalog gallery images with offer overlays using Sharp/Canvas |
| 2 | [meta-feed-sync](skills/meta-feed-sync.SKILL.md) | Sync product catalog feeds to Meta Commerce Manager via API |
| 3 | [dpa-campaign-launcher](skills/dpa-campaign-launcher.SKILL.md) | Launch Dynamic Product Ad campaigns on Meta Ads |
| 4 | [collection-ads-builder](skills/collection-ads-builder.SKILL.md) | Build collection ad campaigns with hero images on Meta |
| 5 | [ugc-campaign-creator](skills/ugc-campaign-creator.SKILL.md) | Create UGC video ad campaigns from a content library |
| 6 | [shopify-collection-sorter](skills/shopify-collection-sorter.SKILL.md) | Sort Shopify collections by sales velocity or stock depth |
| 7 | [price-engine](skills/price-engine.SKILL.md) | Apply percentage discounts and manage compare-at pricing |
| 8 | [variant-mapper](skills/variant-mapper.SKILL.md) | Map Shopify product variants for catalog operations |
| 9 | [feed-registry-manager](skills/feed-registry-manager.SKILL.md) | Track which products have been processed in the feed pipeline |
| 10 | [cdn-image-uploader](skills/cdn-image-uploader.SKILL.md) | Upload product images to Shopify CDN via the Files API |
| 11 | [meta-budget-optimizer](skills/meta-budget-optimizer.SKILL.md) | Analyze and rebalance Meta campaign budgets based on ROAS |
| 12 | [theme-asset-deployer](skills/theme-asset-deployer.SKILL.md) | Deploy Liquid snippets and section files to Shopify themes |
| 13 | [product-tagger](skills/product-tagger.SKILL.md) | Bulk add/remove tags on Shopify products via GraphQL |
| 14 | [metafield-writer](skills/metafield-writer.SKILL.md) | Set custom metafields on products and collections |
| 15 | [size-availability-auditor](skills/size-availability-auditor.SKILL.md) | Audit variant stock levels across size ranges |
| 16 | [webhook-manager](skills/webhook-manager.SKILL.md) | Register and manage Shopify webhooks |
| 17 | [collection-product-set-sync](skills/collection-product-set-sync.SKILL.md) | Sync Shopify collections to Meta product sets for DPA targeting |

## About

These skills accompany **The Agentic Ecom Playbook** by Mazin Zaki.
Powered by Meska AI.
