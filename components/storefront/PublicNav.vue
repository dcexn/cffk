<template>
  <nav class="flex shrink-0 items-center" :aria-label="messages.nav.main">
    <div class="hidden items-center gap-3 sm:flex">
      <template v-if="supportContacts.length">
        <DropdownMenu v-if="supportContacts.length > 1">
          <DropdownMenuTrigger as-child>
            <Button type="button" variant="outline" size="sm">
              <HeadphonesIcon data-icon="inline-start" />
              {{ messages.storefront.contactSupport }}
            </Button>
          </DropdownMenuTrigger>
          <DropdownMenuContent align="end" class="min-w-44">
            <DropdownMenuItem v-for="contact in supportContacts" :key="`${contact.label}-${contact.href}`" as-child>
              <a :href="contact.href" target="_blank" rel="noopener noreferrer">{{ contact.label }}</a>
            </DropdownMenuItem>
          </DropdownMenuContent>
        </DropdownMenu>
        <Button v-else size="sm" variant="outline" as-child>
          <a :href="supportContacts[0].href" target="_blank" rel="noopener noreferrer">
            <HeadphonesIcon data-icon="inline-start" />
            {{ supportContacts[0].label }}
          </a>
        </Button>
      </template>
      <template v-if="user">
        <Button size="sm" variant="outline" as-child><a href="/account/order"><ClipboardListIcon data-icon="inline-start" />{{ messages.nav.myOrders }}</a></Button>
      </template>
      <template v-else>
        <Button size="sm" variant="outline" as-child><a href="/order"><ClipboardListIcon data-icon="inline-start" />{{ messages.nav.orderLookup }}</a></Button>
      </template>
      <template v-if="user">
        <DropdownMenu>
          <DropdownMenuTrigger as-child>
            <Button type="button" variant="ghost" size="icon" class="rounded-full" :aria-label="t(messages.nav.accountMenu, { name: currentUserName })">
              <Avatar class="size-8"><AvatarFallback>{{ userInitials }}</AvatarFallback></Avatar>
            </Button>
          </DropdownMenuTrigger>
          <DropdownMenuContent align="end" :side-offset="8" class="w-48">
            <DropdownMenuLabel>{{ currentUserName }}</DropdownMenuLabel>
            <DropdownMenuSeparator />
            <DropdownMenuGroup>
              <DropdownMenuItem as-child><a href="/account"><UserRoundCogIcon data-icon="inline-start" />{{ messages.nav.accountSettings }}</a></DropdownMenuItem>
              <DropdownMenuItem :disabled="signingOut" @select="signOut"><LogOutIcon data-icon="inline-start" />{{ signingOut ? messages.nav.signingOut : messages.nav.signOut }}</DropdownMenuItem>
            </DropdownMenuGroup>
          </DropdownMenuContent>
        </DropdownMenu>
      </template>
    </div>

    <DropdownMenu>
      <DropdownMenuTrigger as-child>
        <Button type="button" variant="outline" size="icon" class="sm:hidden" :aria-label="messages.nav.menu">
          <MenuIcon />
        </Button>
      </DropdownMenuTrigger>
      <DropdownMenuContent align="end" class="min-w-48">
        <template v-if="supportContacts.length">
          <DropdownMenuItem v-for="contact in supportContacts" :key="`${contact.label}-${contact.href}`" as-child>
            <a :href="contact.href" target="_blank" rel="noopener noreferrer"><HeadphonesIcon data-icon="inline-start" />{{ contact.label }}</a>
          </DropdownMenuItem>
          <DropdownMenuSeparator />
        </template>
        <DropdownMenuItem v-if="user" as-child><a href="/account/order"><ClipboardListIcon data-icon="inline-start" />{{ messages.nav.myOrders }}</a></DropdownMenuItem>
        <DropdownMenuItem v-else as-child><a href="/order"><ClipboardListIcon data-icon="inline-start" />{{ messages.nav.orderLookup }}</a></DropdownMenuItem>
        <template v-if="user">
          <DropdownMenuSeparator />
          <DropdownMenuLabel>{{ currentUserName }}</DropdownMenuLabel>
          <DropdownMenuItem as-child><a href="/account"><UserRoundCogIcon data-icon="inline-start" />{{ messages.nav.accountSettings }}</a></DropdownMenuItem>
          <DropdownMenuItem :disabled="signingOut" @select="signOut"><LogOutIcon data-icon="inline-start" />{{ signingOut ? messages.nav.signingOut : messages.nav.signOut }}</DropdownMenuItem>
        </template>
      </DropdownMenuContent>
    </DropdownMenu>
  </nav>
</template>

<script setup lang="ts">
import { computed, ref } from "vue";
import { ClipboardListIcon, HeadphonesIcon, LogOutIcon, MenuIcon, UserRoundCogIcon } from "@lucide/vue";

import { usePageContext } from "vike-vue/usePageContext";
import { toast } from "vue-sonner";
import { Avatar, AvatarFallback } from "@/components/ui/avatar";
import { Button } from "@/components/ui/button";
import { DropdownMenu, DropdownMenuContent, DropdownMenuGroup, DropdownMenuItem, DropdownMenuLabel, DropdownMenuSeparator, DropdownMenuTrigger } from "@/components/ui/dropdown-menu";
import { authClient } from "@/lib/auth-client";
import { pickLocalizedText, type LocalizedText } from "@/lib/i18n/localized-text";
import { useStorefrontPreferences } from "@/lib/storefront-preferences";

type PublicUser = { id: string; name?: string | null; email?: string | null };
type Site = { supportContact: LocalizedText };
type SupportContact = { label: string; href: string };

const pageContext = usePageContext() as ReturnType<typeof usePageContext> & { user?: PublicUser | null; site: Site };
const user = pageContext.user ?? null;
const site = pageContext.site;
const currentUserName = computed(() => user?.name?.trim() || user?.email || messages.value.nav.account);
const userInitials = computed(() => currentUserName.value.slice(0, 1).toUpperCase());
const signingOut = ref(false);
const { messages, locale, t } = useStorefrontPreferences();
const supportContacts = computed<SupportContact[]>(() => pickLocalizedText(site.supportContact, locale.value)
  .split("\n")
  .map((line) => line.trim())
  .filter(Boolean)
  .map((line) => {
    const separatorIndex = line.indexOf("|");
    const label = separatorIndex >= 0 ? line.slice(0, separatorIndex).trim() : line;
    const address = separatorIndex >= 0 ? line.slice(separatorIndex + 1).trim() : line;
    if (!label || !address) return null;
    return { label, href: address.includes(":") ? address : `mailto:${address}` };
  })
  .filter((contact): contact is SupportContact => contact !== null));

async function signOut() {
  if (signingOut.value) return;
  signingOut.value = true;
  try {
    const result = await authClient.signOut();
    if (result.error) {
      toast.error(messages.value.nav.signOutFailed);
      return;
    }
    toast.success(messages.value.nav.signedOut);
    window.location.assign("/");
  } catch {
    toast.error(messages.value.nav.signOutFailed);
  } finally {
    signingOut.value = false;
  }
}
</script>
