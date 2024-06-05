<script setup>
import { Button } from "@/components/ui/button";

let tutor = [
  "Langkah pertama klik DO IT pada halaman home untuk masuk ke halaman perhitungan prediksi. ",
  "Kedua, setelah masuk ke halaman perhitungan, masukkan file excel yang diminta pada halaman prediksi",
  "Ketiga, klik hitung g-score untuk mengetahui prediksi atau kondisi perusahaan.",
  "Keempat, setelah mengetahui kondisi perusahaan klik uji KNN untuk menguji atau memastikan hasil uji benar valid.",
  //   "Kelima, setelah mengetahui hasil dari uji KNN, anda dapat menyimpan file dalam bentuk pdf dengan mengklik download.",
];
</script>
<template>
  <div>
    <div class="container mx-auto px-6 py-16 pt-28 text-center">
      <div class="mx-auto max-w-[1024px] px-4">
        <h1
          class="text-3xl font-bold text-gray-800 dark:text-white md:text-4xl"
        >
          SISTEM PREDIKSI INSOLVENCY
        </h1>

        <p class="mt-6 text-gray-500 dark:text-gray-300">
          Aplikasi ini merupakan inovasi untuk mempermudah perusahaan mengetahui
          kondisi kesehatan perusahaannya. Dimana pada aplikasi ini akan
          mengelompokkan perusahaan kedalam suatu kondisi bangkrut atau tidak
          bangkrut sesuai dengan algoritma yang digunakan dalam aplikasi ini.
        </p>

        <div class="text-center pt-6">
          <RouterLink to="/hitung">
            <Button class="px-10">Do it</Button></RouterLink
          >
        </div>
      </div>
    </div>

    <div class="px-4">
      <div class="text-center">
        <h1 class="text-xl font-bold">SISTEM PREDIKSI INSOLVENCY</h1>
      </div>
      <div class="mx-auto max-w-lg pt-4">
        <div v-for="(item, index) in tutor" class="py-1">
          {{ index + 1 }} .{{ item }}
        </div>
      </div>
    </div>
  </div>
</template>
