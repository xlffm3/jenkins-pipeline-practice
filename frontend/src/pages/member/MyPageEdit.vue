<template>
  <v-sheet class="d-flex flex-column justify-center mt-12">
    <div class="d-flex justify-center relative">
      <v-card class="card-border px-3 pt-3 pb-5" width="400">
        <v-form ref="memberEditForm" v-model="valid" @submit.prevent>
          <v-card-title class="font-weight-bold justify-center">
            나의 정보 수정
          </v-card-title>
          <v-card-text class="px-4 pt-4 pb-0">
            <div class="d-flex">
              <v-text-field
                  v-model="editingMember.email"
                  :rules="rules.member.email"
                  color="grey darken-1"
                  dense
                  label="이메일을 입력해주세요."
                  outlined
                  prepend-inner-icon="mdi-email"
              ></v-text-field>
            </div>
            <div class="d-flex mt-2">
              <v-text-field
                  v-model="editingMember.age"
                  :rules="rules.member.age"
                  color="grey darken-1"
                  dense
                  label="나이를 입력해주세요."
                  outlined
                  prepend-inner-icon="mdi-account"
              ></v-text-field>
            </div>
            <div class="d-flex mt-2">
              <v-text-field
                  v-model="editingMember.password"
                  :rules="rules.member.password"
                  color="grey darken-1"
                  dense
                  label="비밀번호를 입력해주세요."
                  outlined
                  prepend-inner-icon="mdi-lock"
                  type="password"
              ></v-text-field>
            </div>
            <div class="d-flex mt-2">
              <v-text-field
                  v-model="editingMember.confirmPassword"
                  :rules="[
                  (editingMember.password &&
                    editingMember.password === editingMember.confirmPassword) ||
                    '비밀번호가 일치하지 않습니다.',
                ]"
                  color="grey darken-1"
                  dense
                  label="비밀번호를 한번 더 입력해주세요."
                  outlined
                  prepend-inner-icon="mdi-lock"
                  type="password"
              ></v-text-field>
            </div>
          </v-card-text>
          <v-card-actions class="px-4 pb-4">
            <v-spacer></v-spacer>
            <v-btn text @click="$router.go(-1)">
              취소
            </v-btn>
            <v-btn
                :disabled="!valid"
                color="amber"
                depressed
                @click.prevent="onEditMember"
            >
              확인
            </v-btn>
          </v-card-actions>
        </v-form>
      </v-card>
    </div>
  </v-sheet>
</template>

<script>
import {mapGetters, mapMutations} from "vuex";
import {SET_ACCESS_TOKEN, SET_MEMBER, SHOW_SNACKBAR} from "../../store/shared/mutationTypes";
import {SNACKBAR_MESSAGES} from "../../utils/constants";
import validator from "../../utils/validator";

export default {
  name: "MypageEdit",
  computed: {
    ...mapGetters(["member", "accessToken"]),
  },
  created() {
    const {email, age} = this.member;
    this.editingMember = {
      email,
      age,
      password: "",
      confirmPassword: "",
    };
  },
  methods: {
    ...mapMutations([SHOW_SNACKBAR, SET_MEMBER, SET_ACCESS_TOKEN]),
    isValid() {
      return this.$refs.memberEditForm.validate();
    },
    async onEditMember() {
      const {email, age, password} = this.editingMember;

      try {
        // TODO member 정보를 update하는 API를 추가해주세요
        await fetch("http://localhost:8080/members/me", {
          method: "PUT",
          headers: {
            "Content-Type": "application/json; charset=UTF-8",
            "Authorization": `Bearer ${localStorage.getItem("token")}`,
          },
          body: JSON.stringify({
                email, age, password
              }
          )
        });
        const member = await fetch("http://localhost:8080/members/me", {
          method: "GET",
          headers: {
            "Authorization": `Bearer ${localStorage.getItem("token")}`
          }
        }).then(response => response.json());
        this.setMember(member);
        this.showSnackbar(SNACKBAR_MESSAGES.MEMBER.EDIT.SUCCESS);
        await this.$router.replace("/mypage");
      } catch (e) {
        this.showSnackbar(SNACKBAR_MESSAGES.MEMBER.EDIT.FAIL);
        throw new Error(e);
      }
    },
  },
  data() {
    return {
      editingMember: {},
      valid: false,
      rules: {...validator},
    };
  },
};
</script>
