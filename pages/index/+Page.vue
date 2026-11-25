<template>
  <main class="flex min-h-screen flex-col bg-muted/30">
    <header class="fixed inset-x-0 top-0 z-50 border-b bg-background/95 backdrop-blur">
      <div class="mx-auto flex min-h-16 max-w-6xl items-center justify-between gap-4 px-5">
        <a href="/" class="flex min-w-0 items-center gap-2 font-semibold">
          <img :src="brandLogoUrl" :alt="`${site.name} Logo`" class="size-9 shrink-0 rounded-md object-contain" />
          <span class="truncate">{{ site.name }}</span>
        </a>
        <div class="flex shrink-0 items-center gap-2">
          <PublicNav />
        </div>
      </div>
    </header>

    <section class="border-b bg-background pt-16">
      <div class="mx-auto grid max-w-6xl gap-5 px-5 py-8 sm:grid-cols-[minmax(0,1fr)_22rem] sm:items-center">
        <div v-if="localizedNotice" class="flex min-w-0 items-center gap-3 text-sm leading-6 text-muted-foreground">
          <InfoIcon class="size-4 shrink-0 text-foreground" />
          <div class="min-w-0 max-w-[28.8rem] overflow-hidden whitespace-nowrap">
            <p v-if="!shouldMarqueeNotice" class="m-0 truncate" :title="localizedNotice">{{ localizedNotice }}</p>
            <template v-else>
              <p class="sr-only">{{ localizedNotice }}</p>
              <div aria-hidden="true" class="notice-marquee-track inline-flex w-max animate-notice-marquee">
                <span class="shrink-0 pr-12">{{ localizedNotice }}</span>
                <span class="shrink-0 pr-12">{{ localizedNotice }}</span>
              </div>
            </template>
          </div>
        </div>
        <p v-else class="text-sm text-muted-foreground">{{ localizedSubtitle || messages.storefront.fallbackSubtitle }}</p>
        <div class="relative">
          <SearchIcon class="pointer-events-none absolute left-3 top-1/2 size-4 -translate-y-1/2 text-muted-foreground" />
          <Input v-model="query" class="pl-9" :placeholder="messages.storefront.searchProducts" :aria-label="messages.storefront.searchProducts" />
        </div>
      </div>
    </section>

    <section class="mx-auto w-full max-w-6xl flex-1 px-5 py-10 sm:py-12">
      <div class="flex flex-wrap items-end justify-between gap-4">
        <div>
          <h2 class="text-xl font-semibold tracking-normal">{{ messages.storefront.products }}</h2>
          <p class="mt-1 text-sm text-muted-foreground">{{ query ? t(messages.storefront.relatedProducts, { count: visibleProducts.length }) : messages.storefront.fallbackSubtitle }}</p>
        </div>
        <div v-if="data.categories.length" class="hidden max-w-full gap-2 overflow-x-auto pb-1 sm:flex" :aria-label="messages.storefront.categories">
          <Button size="sm" :variant="selectedCategory === null ? 'default' : 'outline'" @click="selectedCategory = null">{{ messages.storefront.all }}</Button>
          <Button
            v-for="category in data.categories"
            :key="category.id"
            size="sm"
            :variant="selectedCategory === category.id ? 'default' : 'outline'"
            @click="selectedCategory = category.id"
          >
            {{ categoryLabel(category) }}
          </Button>
        </div>
      </div>

      <div v-if="data.categories.length" class="mt-4 sm:hidden">
        <Select :model-value="selectedCategory === null ? 'all' : String(selectedCategory)" @update:model-value="onCategoryChange">
          <SelectTrigger class="w-full" :aria-label="messages.storefront.categories">
            <SelectValue />
          </SelectTrigger>
          <SelectContent>
            <SelectItem value="all">{{ messages.storefront.all }}</SelectItem>
            <SelectItem v-for="category in data.categories" :key="category.id" :value="String(category.id)">{{ categoryLabel(category) }}</SelectItem>
          </SelectContent>
        </Select>
      </div>

      <div v-if="groupedProducts.length" class="mt-6 space-y-8">
        <section v-for="group in groupedProducts" :key="group.category?.id || 'none'" class="space-y-3">
          <Badge class="px-3 py-1 text-lg font-bold bg-[#45c3d9] hover:bg-[#45c3d9]/90 text-white rounded-md border-none tracking-wider">{{ group.category ? categoryLabel(group.category) : messages.storefront.products }}</Badge>

          <div class="hidden overflow-hidden rounded-lg border bg-card shadow-sm sm:block">
            <Table>
              <TableHeader>
                <TableRow>
                  <TableHead class="h-11 text-sm font-medium text-foreground">{{ messages.storefront.columnProduct }}</TableHead>
                  <TableHead class="h-11 w-32 text-sm font-medium text-foreground">{{ messages.storefront.columnStock }}</TableHead>
                  <TableHead class="h-11 w-40 text-sm font-medium text-foreground">{{ messages.storefront.columnPrice }}</TableHead>
                  <TableHead class="h-11 w-28 text-center text-sm font-medium text-foreground">{{ messages.storefront.columnAction }}</TableHead>
                </TableRow>
              </TableHeader>
              <TableBody>
                <TableRow v-for="product in group.products" :key="product.id">
                  <TableCell class="py-4"><a :href="`/product/${product.slug}`" class="text-base font-normal hover:text-[#45c3d9]">{{ productName(product) }}</a></TableCell>
                  <TableCell class="py-4"><span :class="stockClass(product)" class="text-base font-medium">{{ stockLabel(product) }}</span></TableCell>
                  <TableCell class="py-4 text-base font-bold tabular-nums">{{ product.price }}<CurrencyHint /></TableCell>
                  <TableCell class="py-4 text-right">
                    <Button as-child size="sm" variant="outline" class="rounded-[4px] text-sm border-[#45c3d9] text-[#45c3d9] hover:bg-[#45c3d9] hover:text-white">
                      <a :href="`/product/${product.slug}`">{{ messages.storefront.viewDetails }}</a>
                    </Button>
                  </TableCell>
                </TableRow>
              </TableBody>
            </Table>
          </div>

          <div class="grid gap-4 sm:hidden">
            <article v-for="product in group.products" :key="product.id" class="rounded-lg border bg-card p-4 shadow-sm">
              <a :href="`/product/${product.slug}`" class="block">
                <h3 class="break-words text-base font-medium tracking-normal">{{ productName(product) }}</h3>
                <p v-if="productSubtitle(product)" class="mt-1 break-words text-sm text-muted-foreground">{{ productSubtitle(product) }}</p>
              </a>
              <div class="mt-4 flex items-center justify-between gap-2">
                <span :class="stockClass(product)" class="rounded-full bg-muted px-2.5 py-1 text-xs">{{ stockLabel(product) }}</span>
                <span class="shrink-0 text-lg font-bold tabular-nums">{{ product.price }}<CurrencyHint /></span>
              </div>
              <Button as-child size="sm" variant="outline" class="mt-4 w-full rounded-[4px] text-sm border-[#45c3d9] text-[#45c3d9] hover:bg-[#45c3d9] hover:text-white">
                <a :href="`/product/${product.slug}`">{{ messages.storefront.viewDetails }}</a>
              </Button>
            </article>
          </div>
        </section>
      </div>

      <div v-else class="mt-6 border border-dashed bg-background px-6 py-16 text-center">
        <PackageOpenIcon class="mx-auto size-8 text-muted-foreground" />
        <h3 class="mt-4 text-base font-semibold">{{ messages.storefront.noProducts }}</h3>
        <p class="mt-2 text-sm text-muted-foreground">{{ messages.storefront.noProductsDescription }}</p>
      </div>
    </section>

    <StorefrontFooter />

    <Dialog :open="noticePopupOpen" @update:open="closeNoticePopup">
      <DialogContent class="max-w-lg">
        <DialogHeader>
          <DialogTitle>{{ messages.storefront.noticeTitle }}</DialogTitle>
        </DialogHeader>
        <p class="whitespace-pre-wrap text-sm leading-6 text-muted-foreground">{{ localizedNotice }}</p>
        <DialogFooter>
          <DialogClose as-child><Button type="button" class="w-full">{{ messages.common.close }}</Button></DialogClose>
        </DialogFooter>
      </DialogContent>
    </Dialog>
  </main>
</template>

<script lang="ts" setup>
import { computed, onMounted, ref } from "vue";
import { InfoIcon, PackageOpenIcon, SearchIcon } from "@lucide/vue";
import logoUrl from "@/assets/logo.svg?url";
import { useData } from "vike-vue/useData";
import { usePageContext } from "vike-vue/usePageContext";
import PublicNav from "@/components/storefront/PublicNav.vue";
import StorefrontFooter from "@/components/storefront/StorefrontFooter.vue";
import { Badge } from "@/components/ui/badge";
import { Button } from "@/components/ui/button";
import { Dialog, DialogClose, DialogContent, DialogFooter, DialogHeader, DialogTitle } from "@/components/ui/dialog";
import { Input } from "@/components/ui/input";
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from "@/components/ui/select";
import { Table, TableBody, TableCell, TableHead, TableHeader, TableRow } from "@/components/ui/table";
import type { Data } from "./+data.server";
import CurrencyHint from "@/components/storefront/CurrencyHint.vue";
import { useStorefrontPreferences } from "@/lib/storefront-preferences";
import { pickLocalizedOptionalText, pickLocalizedText, type LocalizedNullableText } from "@/lib/i18n/localized-text";
import { readCookieValue } from "@/lib/i18n/locale";
import { noticeVersion } from "@/lib/notice-version";

type Product = Data["products"][number];
type Category = Data["categories"][number];
type Site = {
  name: string;
  subtitle: LocalizedNullableText;
  logo: string | null;
  notice: LocalizedNullableText;
  noticePopupEnabled: boolean;

  footerText: LocalizedNullableText;
};

const data = useData<Data>();
const pageContext = usePageContext() as unknown as { site: Site };
const site = pageContext.site;
const { messages, t, locale } = useStorefrontPreferences();
const selectedCategory = ref<number | null>(null);
const query = ref("");
const localizedNotice = computed(() => (site.notice ? pickLocalizedText(site.notice, locale.value) : ""));
const localizedSubtitle = computed(() => (site.subtitle ? pickLocalizedText(site.subtitle, locale.value) : ""));
const shouldMarqueeNotice = computed(() => localizedNotice.value.length > 30);
const noticeCookieName = "storefront-notice-seen";
const noticePopupOpen = ref(false);

function writeNoticeSeen() {
  const text = localizedNotice.value.trim();
  if (!text || typeof document === "undefined") return;
  const secure = window.location.protocol === "https:" ? "; secure" : "";
  document.cookie = `${noticeCookieName}=${noticeVersion(text)}; path=/; max-age=31536000; samesite=lax${secure}`;
}
function closeNoticePopup(open: boolean) {
  noticePopupOpen.value = open;
  if (!open) writeNoticeSeen();
}
onMounted(() => {
  const text = localizedNotice.value.trim();
  if (!site.noticePopupEnabled || !text) return;
  if (readCookieValue(document.cookie, noticeCookieName) !== noticeVersion(text)) noticePopupOpen.value = true;
});
const brandLogoUrl = computed(() => site.logo || logoUrl);
function productName(product: Product) { return pickLocalizedText(product.name, locale.value); }
function productSubtitle(product: Product) { return product.subtitle ? pickLocalizedOptionalText(product.subtitle, locale.value) : null; }
function categoryLabel(category: Category) { return pickLocalizedText(category.name, locale.value); }
function onCategoryChange(value: unknown) {
  if (typeof value !== "string") return;
  selectedCategory.value = value === "all" ? null : Number(value);
}
const visibleProducts = computed(() => {
  const keyword = query.value.trim().toLowerCase();
  return data.products.filter((product) =>
    (selectedCategory.value === null || product.categoryId === selectedCategory.value)
    && (!keyword || productName(product).toLowerCase().includes(keyword) || (productSubtitle(product) ?? "").toLowerCase().includes(keyword)),
  );
});

const groupedProducts = computed(() => {
  const groups: { category: Category | null; products: Product[] }[] = [];

  const categoriesToShow = selectedCategory.value !== null
    ? data.categories.filter((category) => category.id === selectedCategory.value)
    : data.categories;

  for (const category of categoriesToShow) {
    const products = visibleProducts.value.filter((product) => product.categoryId === category.id);
    if (products.length > 0) {
      groups.push({ category, products });
    }
  }

  if (selectedCategory.value === null) {
    const uncategorizedProducts = visibleProducts.value.filter((product) => !product.categoryId);
    if (uncategorizedProducts.length > 0) {
      groups.push({ category: null, products: uncategorizedProducts });
    }
  }

  return groups;
});

function stockLabel(product: Product) {
  if (product.availableStock === null) return messages.value.storefront.stockAvailable;
  return product.availableStock > 0
    ? t(messages.value.storefront.stockCount, { count: product.availableStock })
    : messages.value.storefront.outOfStock;
}
function stockClass(product: Product) {
  return product.availableStock === 0 ? "text-orange-500" : "text-muted-foreground";
}
</script>

<style scoped>
@keyframes notice-marquee {
  from {
    transform: translateX(0);
  }

  to {
    transform: translateX(-50%);
  }
}

.animate-notice-marquee {
  animation: notice-marquee 18s linear infinite;
}

@media (prefers-reduced-motion: reduce) {
  .animate-notice-marquee {
    animation: none;
  }
}
</style>
