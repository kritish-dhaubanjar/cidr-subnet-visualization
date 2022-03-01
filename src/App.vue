<template>
  <a
    class="github-fork-ribbon"
    href="https://github.com/kritish-dhaubanjar/cidr-subnet-visualization"
    data-ribbon="Fork me on GitHub"
    title="Fork me on GitHub"
    >Fork me on GitHub</a
  >

  <div class="container my-5">
    <div class="row">
      <div class="col-12">
        <h1>IPv4 CIDR Notation</h1>

        <p>
          A system called
          <a
            target="_blank"
            href="https://www.digitalocean.com/community/tutorials/understanding-ip-addresses-subnets-and-cidr-notation-for-networking"
          >
            Classless Inter-Domain Routing,</a
          >
          or CIDR, was developed as an alternative to traditional subnetting.
          The idea is that you can add a specification in the IP address itself
          as to the number of significant bits that make up the routing or
          networking portion.
          <br />
          <br />
          For example, we could express the idea that the IP address
          <span class="fw-bold">192.168.0.15</span> is associated with the
          netmask <span class="fw-bold">255.255.255.0</span> by using the CIDR
          notation of <span class="fw-bold">192.168.0.15/24</span>. This means
          that the first 24 bits of the IP address given are considered
          significant for the network routing.
        </p>

        <em
          >CIDR allows us more control over addressing continuous blocks of IP
          addresses. This is much more useful than the subnetting using IPv4
          Addresses Classes and Reserved Ranges.</em
        >
      </div>
    </div>

    <div class="row mt-5 mb-4">
      <div
        class="col-12 input-group justify-content-center position-relative ip-address"
      >
        <template v-for="i in 4" :key="i">
          <input
            :value="ipv4[i - 1]"
            @keypress="validate"
            @input="(e) => setIPv4Segment(e, i - 1)"
            min="0"
            max="255"
            type="number"
            class="form-control text-center"
          />
          <h1 v-show="i < 4" class="px-3">.</h1>
        </template>

        <h1 class="px-3">/</h1>
        <input
          v-model="cidr"
          @keypress="validate"
          min="0"
          max="32"
          type="number"
          class="form-control text-center"
        />
      </div>
    </div>

    <div class="row">
      <div class="col-12">
        <h4 class="mt-4 mb-3">IPv4 Address</h4>
      </div>

      <div class="col-3 segment" v-for="(mask, i) in ipv4" :key="i">
        <Segment :position="i" :mask="mask" :cidr="cidr" />
      </div>

      <div class="col-12">
        <h4 class="mt-4 mb-3">Subnet Mask</h4>
      </div>

      <div
        class="col-3 segment"
        v-for="(mask, i) in subnet.subnetMask"
        :key="i"
      >
        <Segment :position="i" :mask="mask" :cidr="cidr" />
      </div>
    </div>

    <div class="row">
      <div class="col-12">
        <h4 class="mt-4">Subnet</h4>

        <table class="table">
          <tbody>
            <tr>
              <td scope="col">Total IPv4</td>
              <td scope="col">
                <code>{{ subnet.count }}</code>
              </td>
            </tr>
            <tr>
              <td scope="row">Subnet Mask</td>
              <td>
                <code>{{ subnet.subnetMask.join(".") }}</code>
              </td>
            </tr>
            <tr>
              <td scope="row">Wildcard Mask</td>
              <td>
                <code>{{ subnet.wildcardMask.join(".") }}</code>
              </td>
            </tr>
            <tr>
              <td scope="row">Network</td>
              <td>
                <code>{{ subnet.networkAddress.join(".") }}/{{ cidr }}</code>
              </td>
            </tr>
            <tr>
              <td scope="row">Broadcast Address</td>
              <td>
                <code>{{ subnet.broadcastAddress.join(".") }}</code>
              </td>
            </tr>
            <tr>
              <td scope="row">Host Addresses</td>
              <td>
                <code
                  >{{ subnet.hostAddresses.start.join(".") }} -
                  {{ subnet.hostAddresses.end.join(".") }}</code
                >
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <center class="mt-5">
      <em
        >* For routing mask values &lt;= 30, first and last IPs are base and
        broadcast addresses and are unusable.</em
      >
    </center>
  </div>
</template>

<script setup lang="ts">
import { ref } from "@vue/reactivity";
import { watch } from "@vue/runtime-core";
import { computeSubnet } from "compute-subnet";

import Segment from "./components/Segment.vue";

const NUMBERS = /[0-9/]+/;

const cidr = ref(23);
const ipv4 = ref([192, 168, 100, 128]);
const subnet = ref(computeSubnet(ipv4.value, cidr.value));

function validate(e: KeyboardEvent) {
  if (!NUMBERS.test(e.key)) {
    e.preventDefault();
  }
}

watch(cidr, (newValue: number, oldValue: number) => {
  if (!newValue) {
    cidr.value = 0;
  } else if (newValue > 32) {
    cidr.value = oldValue;
  }

  subnet.value = computeSubnet(ipv4.value, cidr.value);
});

function setIPv4Segment(e: InputEvent, index: number) {
  const valid = [...ipv4.value];
  const data = parseInt((e.target as HTMLInputElement).value);

  if (data >= 0 && data <= 255) {
    valid[index] = data;
  }

  ipv4.value = valid;
  subnet.value = computeSubnet(ipv4.value, cidr.value);
}
</script>

<style lang="scss">
@import "bootstrap";
@import "./assets/global.scss";
</style>
