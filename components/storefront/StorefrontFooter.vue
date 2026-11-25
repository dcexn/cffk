<template>
  <footer>
    <div class="mx-auto max-w-6xl px-5 py-6 text-sm text-muted-foreground">
      <p class="whitespace-pre-wrap">{{ footerText }}</p>
    </div>
  </footer>
</template>

<script lang="ts" setup>
import { computed } from "vue";
import { usePageContext } from "vike-vue/usePageContext";

import { useStorefrontPreferences } from "@/lib/storefront-preferences";
import { pickLocalizedText, type LocalizedText } from "@/lib/i18n/localized-text";

type Site = {
  footerText: LocalizedText;
};

const pageContext = usePageContext() as unknown as { site: Site };
const site = pageContext.site;
const { locale } = useStorefrontPreferences();
const footerText = computed(() => pickLocalizedText(site.footerText, locale.value));
</script>
