<template>
    <Pagination v-if="pageStore.getCommentsMaxPage > 1"
                :key="paginationKey"/>
    <ul class=" d-flex flex-column" style="width:100%; margin-block:0; padding-inline-start: 0">
        <li v-for="comment in commentsStore.getComments"
            :key="comment.id" :id="'comment-' + comment.id">
            <div  v-if="comment.parentId === null" class="d-flex flex-column">
                <div class="d-flex justify-content-between align-items-center">
                    <div class="d-flex align-items-center">
                        <img v-if="comment.commenterImagePath" :src="comment.commenterImagePath"
                             style="width: 2.5rem; height: 2.5em; border-radius: 100%; cursor: pointer;"
                             alt=""
                             @click="router.replace( '/members/'+comment.commenterId)"/>
                        <el-icon v-else size="30"
                                 style="width: 2.5rem;  height:2.5rem; border-radius: 100%; color:white; background-color: #E2E2E2; cursor: pointer;"
                                 @click="router.replace( '/members/'+comment.commenterId)">
                            <Avatar/>
                        </el-icon>
                        <div class="ms-2 d-flex flex-column">
                            <div @click="router.replace( '/members/'+comment.commenterId)" style="cursor: pointer; font-size:1.2rem;">{{ comment.commenterNickname }}</div>
                            <div style="font-size:0.8rem; color:#929292;">{{ comment.createdAt }}</div>
                        </div>
                    </div>
                    <div class="d-flex align-items-center">
                        <div class="d-flex">
                            <el-button class="like_button pe-2 ps-2"
                                       size="default"
                                       @click="onLikeComment(comment.id)"
                                       :disabled="!authStore.isLoggedIn">
                                <el-icon size="18">
                                    <CaretTop />
                                </el-icon>
                                <span style="font-size: 0.9rem;">
                                    {{comment.likeNum}}
                                </span>
                            </el-button>
                            <el-button class="dislike_button pe-2 ps-2"
                                       size="default"
                                       @click="onDislikeComment( comment.id)"
                                       :disabled="!authStore.isLoggedIn">
                                <el-icon size="18">
                                    <CaretBottom />
                                </el-icon>
                                <span style="font-size: 0.9rem;">
                                    {{comment.dislikeNum}}
                                </span>
                            </el-button>
                        </div>
                        <div v-if="isCommenter(comment)" class="d-flex ms-4 pt-1">
                            <el-dropdown  trigger="click">
                                <span class="el-dropdown-link">
                                    <el-icon size="18" color="#B2B2B2" class="dropdown" style="cursor: pointer;">
                                        <MoreFilled/>
                                    </el-icon>
                                </span>
                                <template #dropdown>
                                    <el-dropdown-menu class="dropdown_menu d-flex flex-column"
                                                      style="
                                        --el-dropdown-menuItem-hover-fill: white;
                                        --el-menu-hover-bg-color: white;
                                        --el-menu-hover-text-color: #409eff;
                                        --el-menu-active-color: #409eff">
                                        <el-dropdown-item class="d-flex" style="padding-right: 1rem; padding-left:1rem;" @click="onEditComment()">
                                            <el-icon>
                                                <Edit />
                                            </el-icon>
                                            수정하기
                                        </el-dropdown-item>
                                        <el-dropdown-item class="d-flex" @click="onDeleteComment()">
                                            <el-icon>
                                                <Delete />
                                            </el-icon>
                                            삭제하기
                                        </el-dropdown-item>
                                    </el-dropdown-menu>
                                </template>
                            </el-dropdown>
                        </div>
                    </div>
                </div>
                <div class="mt-1" style="white-space: pre-line; line-height: 2rem;">
                    {{ comment.content }}
                </div>
                <div v-if="!comment.reply"
                     class="write_reply"
                     style="font-size:0.8rem; cursor:pointer"
                     @click="comment.reply = true; setWriteReply(comment)">
                    답글 쓰기
                </div>
                <div v-else>
                    <div class="write_reply"
                         style="font-size:0.8rem; cursor:pointer"
                         @click="comment.reply = false, resetReply">
                        답글 취소
                    </div>
                    <div class="d-flex flex-column p-4">
                        <div class="d-flex justify-content-between" style="width:95%; margin:auto 0 auto auto;">
                            <img v-if="comment.commenterImagePath" :src="comment.commenterImagePath"
                                 style="width: 3rem; height: 3rem; border-radius: 100%;"
                                 alt="">
                            <el-icon v-else size="40"
                                     style="width: 3rem;  height:3rem; border-radius: 100%; color:white; background-color: #E2E2E2;">
                                <Avatar/>
                            </el-icon>
                            <el-input v-model="reply.content"
                                      class="comment-input ms-4"
                                      size="large"
                                      :autosize="{ minRows: 3, maxRows: 6 }"
                                      placeholder="댓글을 입력해주세요."
                                      show-word-limit
                                      maxlength="3000"
                                      type="textarea"
                                      resize="none"
                                      input-style="font-size: 1rem; line-height: 2rem; letter-spacing:0.05rem;">
                            </el-input>
                        </div>
                        <el-button class="comment-button"
                                   type="primary"
                                   @click="onWriteComment(reply)"
                                   size="default"
                                   :disabled="reply.content.length<1 || !authStore.isLoggedIn"
                                   style="max-width: 5rem; margin: 1.5rem 0 auto auto;">
                            답글 쓰기
                        </el-button>
                    </div>
                </div>
            </div>
            <div v-else>
                <div class="d-flex flex-column" style="width:95%; margin: 0 0 auto auto;">
                    <div class="d-flex justify-content-between align-items-center">
                        <div class="d-flex align-items-center">
                            <img v-if="comment.commenterImagePath" :src="comment.commenterImagePath"
                                 style="width: 2.5rem; height: 2.5em; border-radius: 100%; cursor: pointer;"
                                 alt=""
                                 @click="router.replace( '/members/'+comment.commenterId)"/>
                            <el-icon v-else size="30"
                                     style="width: 2.5rem;  height:2.5rem; border-radius: 100%; color:white; background-color: #E2E2E2; cursor: pointer;"
                                     @click="router.replace( '/members/'+comment.commenterId)">
                                <Avatar/>
                            </el-icon>
                            <div class="ms-2 d-flex flex-column">
                                <div @click="router.replace( '/members/'+comment.commenterId)" style="cursor: pointer; font-size:1.2rem;">{{ comment.commenterNickname }}</div>
                                <div style="font-size:0.8rem; color:#929292;">{{ comment.createdAt }}</div>
                            </div>
                        </div>
                        <div class="d-flex align-items-center">
                            <div class="d-flex">
                                <el-button class="like_button pe-2 ps-2"
                                           size="default"
                                           @click="onLikeComment(comment.id)"
                                           :disabled="!authStore.isLoggedIn">
                                    <el-icon size="18">
                                        <CaretTop />
                                    </el-icon>
                                    <span style="font-size: 0.9rem;">
                                     {{comment.likeNum}}
                                 </span>
                                </el-button>
                                <el-button class="dislike_button pe-2 ps-2"
                                           size="default"
                                           @click="onDislikeComment(comment.id)"
                                           :disabled="!authStore.isLoggedIn">
                                    <el-icon size="18">
                                        <CaretBottom />
                                    </el-icon>
                                    <span style="font-size: 0.9rem;">
                                    {{comment.dislikeNum}}
                                 </span>
                                </el-button>
                            </div>
                            <div v-if="isCommenter(comment)" class="d-flex ms-4 pt-1">
                                <el-dropdown  trigger="click">
                                    <span class="el-dropdown-link">
                                        <el-icon size="18" color="#B2B2B2" class="dropdown" style="cursor: pointer;">
                                            <MoreFilled/>
                                        </el-icon>
                                    </span>
                                    <template #dropdown>
                                        <el-dropdown-menu class="dropdown_menu d-flex flex-column"
                                                          style="
                                --el-dropdown-menuItem-hover-fill: white;
                                --el-menu-hover-bg-color: white;
                                --el-menu-hover-text-color: #409eff;
                                --el-menu-active-color: #409eff">
                                            <el-dropdown-item class="d-flex" style="padding-right: 1rem; padding-left:1rem;" @click="onEditComment()">
                                                <el-icon>
                                                    <Edit />
                                                </el-icon>
                                                수정하기
                                            </el-dropdown-item>
                                            <el-dropdown-item class="d-flex" @click="onDeleteComment()">
                                                <el-icon>
                                                    <Delete />
                                                </el-icon>
                                                삭제하기
                                            </el-dropdown-item>
                                        </el-dropdown-menu>
                                    </template>
                                </el-dropdown>
                            </div>
                        </div>
                    </div>
                    <div class="mt-1" style="white-space: pre-line; line-height: 2rem;">
                        <span v-if="comment.targetNickname !== null" class="target_nickname d-inline" @click="router.replace('/members/'+comment.targetMemberId)">@{{comment.targetNickname}}</span><span style="display: inline;">{{ comment.content }}</span>
                    </div>
                    <div v-if="!comment.reply"
                         class="write_reply"
                         style="font-size:0.8rem; cursor:pointer"
                         @click="comment.reply = true, reply.content = '@'+comment.commenterNickname+' ', setWriteMentionReply(comment, comment.parentId)">
                        답글 쓰기
                    </div>
                    <div v-else>
                        <div class="write_reply"
                             style="font-size:0.8rem; cursor:pointer"
                             @click="comment.reply = false, resetReply">
                            답글 취소
                        </div>
                        <div class="d-flex flex-column p-4">
                            <div class="d-flex justify-content-between" style="width:95%; margin: auto 0 auto auto;">
                                <img v-if="comment.commenterImagePath" :src="comment.commenterImagePath"
                                     style="width: 3rem; height: 3rem; border-radius: 100%;"
                                     alt="">
                                <el-icon v-else size="40"
                                         style="width: 3rem;  height:3rem; border-radius: 100%; color:white; background-color: #E2E2E2;">
                                    <Avatar/>
                                </el-icon>
                                <el-input v-model="reply.content"
                                          class="comment-input ms-4"
                                          size="large"
                                          :autosize="{ minRows: 3, maxRows: 6 }"
                                          placeholder="댓글을 입력해주세요."
                                          show-word-limit
                                          maxlength="3000"
                                          type="textarea"
                                          resize="none"
                                          input-style="font-size: 1rem; line-height: 2rem; letter-spacing:0.05rem;">
                                </el-input>
                            </div>
                            <el-button class="comment-button"
                                       type="primary"
                                       @click="onWriteComment(reply)"
                                       size="default"
                                       :disabled="reply.content.length<1 || !authStore.isLoggedIn"
                                       style="max-width: 5rem; margin: 1.5rem 0 auto auto;">
                                답글 쓰기
                            </el-button>
                        </div>
                    </div>
                </div>
            </div>
            <el-divider v-if=" commentsStore.hasChild(comment)" border-style="hidden" class="mt-3 mb-3"/>
            <el-divider v-else-if="!commentsStore.isLastComment(comment) && comment.parentId !== null && !commentsStore.isLastChild(comment)" border-style="dashed" class="mt-3 mb-3" style="width:95%; margin: 0 0 auto auto;"/>
            <el-divider v-else class="mt-3 mb-3"/>
        </li>
    </ul>
    <Pagination v-if="pageStore.getCommentsMaxPage > 1" :key="paginationKey"/>
</template>
<script setup lang="ts">
import {useCommentsStore} from "@/stores/comments";
import {useRoute, useRouter} from "vue-router";
import Pagination from "@/components/pagination/PaginationComponent.vue";
import {usePageStore} from "@/stores/page";
import {useAuthStore} from "@/stores/auth";
import type {Comment} from "@/custom-types/comment";
import {Avatar, CaretBottom, CaretTop, MoreFilled} from "@element-plus/icons-vue";
import {dislikeComment, likeComment} from "@/controller/api/comment";
import {useResponseStore} from "@/stores/Response";
import {inject, onMounted, ref} from "vue";

const commentsStore = useCommentsStore();
const pageStore = usePageStore();
const authStore = useAuthStore();
const resStore = useResponseStore();
const router = useRouter();
const route = useRoute();
const paginationKey = ref(0);

onMounted(async () => {
    paginationKey.value++;
})

const hasChild = function(parent: Comment){
    if(commentsStore.hasChild(parent)){
        console.log("true")
    }
    return commentsStore.hasChild(parent);
}

const isCommenter = function (comment: Comment) {

    if(authStore.isLoggedIn){
        return comment.commenterId === authStore.getId;
    } else {
        return false;
    }
}

const reply = ref({
    content: '',
    parentId: null as null| number,
    targetId: null as null | number,
})

const writeReply = ref({
    parent : null as null|Comment,
    target : null as null|Comment,
})

const setWriteReply = function(parent: Comment) {
    if(writeReply.value.parent !== null) writeReply.value.parent.reply = false;
    else if(writeReply.value.target !== null) writeReply.value.target.reply = false;

    writeReply.value.parent = parent;
    reply.value.parentId = parent.id.valueOf();
    reply.value.targetId = null;
    reply.value.content = '';
}
const setWriteMentionReply = function(target: Comment, parentId: number) {
    if(writeReply.value.parent !== null) writeReply.value.parent.reply = false;
    else if(writeReply.value.target !== null) writeReply.value.target.reply = false;

    writeReply.value.target = target;
    reply.value.parentId = parentId;
    reply.value.targetId = target.id.valueOf();
}

const resetReply = function(){
    reply.value.content = '';
    reply.value.parentId = null;
    reply.value.targetId = null;
}

const onLikeComment = async function ( commentId: number) {
    await likeComment(route.path+'/comments/'+commentId+'/like')
        .then((response:any)=>{
            if(resStore.isOK){
                onGetArticle();
            } else {
                alert(resStore.getErrorMessage);
                console.log(response);
            }
        }).catch(error => {
            alert(error);
            console.log(error)
        })
}

const onDislikeComment = async function (commentId: number) {
    await dislikeComment(route.path+'/comments/'+commentId+'/dislike')
        .then((response:any)=>{
            if(resStore.isOK){
                onGetArticle();
            } else {
                alert(resStore.getErrorMessage);
                console.log(response);
            }
        }).catch(error => {
            alert(error);
            console.log(error)
        })
}

const onEditComment = function(){

}

const onDeleteComment = function(){

}

const onGetArticle: any = inject('onGetArticle');
const onWriteComment: any = inject('onWriteComment');
</script>
<style scoped>
@import "../css/contentBase.css";

.write_reply{
    color:#B2B2B2;
}
.write_reply:hover {
    color: #409eff;
}
.like_button {
    --el-button-hover-bg-color: white;
    border:solid thin #409eff;
    color:#409eff;
}

.dislike_button {
    --el-button-hover-bg-color: white;
    border:solid thin #dc3545;
    color:#dc3545;
}

.like_button:hover, .like_button:active{
    border-color: #409eff;
    background-color: lightcyan;
    color:#409eff;
}

.dislike_button:hover, .dislike_button:active{
    color:#dc3545;
    background-color: #ffe0eb;
    border-color:#dc3545;
}

.target_nickname {
    color:#409eff;
    cursor: pointer;
}

.target_nickname:hover  {
    color:skyblue;
}
</style>