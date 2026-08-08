---
title: '{{ replace .File.ContentBaseName "-" " " | title }}'
date: '{{ .Date }}'
draft: true
# One or two sentences shown under the title and used as the recipe's
# intro — the Markdown body below should contain only Instructions/Tips.
description: ''
# Teaser shown on list pages — usually the same text as description. If
# omitted, Hugo falls back to the start of the body (i.e. instruction text).
summary: ''
prepTime: 15 # minutes
cookTime: 30 # minutes
servings: 4
# Optional — renders a Nutrition Facts card after the recipe and feeds
# schema.org. Per serving; grams, except sodium in mg. Any subset works.
# nutrition:
#   calories: 350
#   protein: 20
#   carbs: 40
#   fat: 12
#   fiber: 4
#   sugar: 8
#   sodium: 600
# Tags cover everything: meal (dinner, breakfast), cuisine (italian),
# effort/diet (quick, vegetarian), and conventions like components or
# meal-prep — they power the home page's filter chips and search.
# Component recipes (dough, sauces, stocks): tag with components and
# reference them from other recipes with the recipe shortcode (see README).
tags: []
# Optional — for batch cooking; renders a Storage & Reheating card.
# All fields free-text and optional.
# mealprep:
#   yields: 8 portions / 4 containers
#   fridge: 4 days
#   freezer: 3 months
#   reheat: Microwave 2–3 min, covered.
# Either a flat list, or grouped into sections (don't mix the two forms):
#   ingredients:
#     - name: Dough
#       items: ['500 g flour', '325 ml water']
#     - name: Sauce
#       items: ['400 g crushed tomatoes']
# For sectioned instructions, use "### Dough" subheadings in the body below.
ingredients:
  - ''
# The cover photo lives next to index.md, named exactly after the folder.
# Other photos: <slug>-<descriptor>.jpg (describe what's shown, not step
# numbers — steps renumber when recipes get edited).
cover:
  image: '{{ .File.ContentBaseName }}.jpg'
  alt: ''
---

## Instructions

1. First step.
2. Second step.

## Tips

- Optional notes, substitutions, storage advice.
