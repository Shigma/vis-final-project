<script>
/**
 * A component that displays important information for a given keyword
 *
 * @author He, Hao
 * @since 2019-1-9
 */

const maildata = require('../dist/mails.json');
const userdata = require('../dist/users.json');
const keywords = require('../dist/keywords.json');
const keywordExtraction = require('./Keyword.js');

const keywordMap = new Map();
keywords.forEach(item => {
    keywordMap.set(item.keyword, item);
});

module.exports = {
    props: ['data'],
    data: () => ({
        startDate: null,
        endDate: null,
    }),
    computed: {
        keyword() {
            return keywordMap.get(this.data.word || 'mac')
        },
        // users is an array, each element has id, name and value field.
        users() {
            // Use precomputed data
            if (this.startDate === null && this.endDate === null)
                return this.keyword.users;

            // Compute on-the-fly
            let result = [];
            let resultIdMap = new Map();
            this.keyword.mails.filter(this.filterWithTime).forEach(id => {
                let currUserId = maildata[id].userId;
                if (currUserId === -1) return;
                let resultId = resultIdMap.get(currUserId);
                if (resultId === undefined) {
                    result.push({
                        id: currUserId,
                        name: userdata[currUserId].name,
                        value: 1,
                    });
                    resultIdMap.set(currUserId, result.length - 1);
                } else {
                    result[resultId].value++;
                }
            });
            result.sort((a, b) => {
                if (a.value > b.value) return -1;
                if (a.vaule < b.value) return 1;
                return 0;
            });
            result = result.filter((a, index) => index <= 30);
            return result;
        },
        // relatedKeywords is an array, each element has name and value field
        relatedKeywords() {
            // Use precomputed data
            if (this.startDate === null && this.endDate === null)
                return this.keyword.relatedKeywords;

            // Compute on-the-fly
            let result = [];
            let resultIdMap = new Map();
            this.keyword.mails.filter(this.filterWithTime).forEach(id => {
                const keys = keywordExtraction.generateKeywords([id]);
                keys.forEach(key => {
                    //console.log(this.data.word);
                    if (key.name.toLowerCase() === this.data.word) return;
                    const resultId = resultIdMap.get(key.name);
                    if (resultId === undefined) {
                        result.push({ name: key.name, value: 1 });
                        resultIdMap.set(key.name, result.length - 1);
                    } else {
                        result[resultId].value++;
                    }
                });
            });
            result.sort((a, b) => {
                if (a.value > b.value) return -1;
                else if (a.value < b.value) return 1;
                return 0;
            });
            return result.filter((word, index) => index <= 15);
        },
    },
    watch: {
        'data.word'() {
            this.$refs.card.showMailList = false
        },
    },
    methods: {
        filterWithTime(mailId) {
            let flag = true;
            let date = maildata[mailId].date;
            if (this.startDate) flag &= date > this.startDate;
            if (this.endDate) flag &= date < this.endDate;
            return flag;
        },
    },
};
</script>

<template>
    <card-view :title="data.word" type="keyword" envelop ref="card">
        <div class="metadata"> Related Mails: {{ keyword.mails.length }}
            <br>Related Users: {{ users.length }}
        </div>
        <line-chart :data="keyword.activity" tag="KeywordOverview" origin="keyword"
            :start-date.sync="startDate" :end-date.sync="endDate"/>
        <word-cloud :data="users" tag="user" origin="keyword"/>
        <bar-chart :data="relatedKeywords" tag="keyword" origin="keyword"/>
        <mail-list slot="mail-list" :mails="keyword.mails" origin="keyword-mail-list"
            :startDate="startDate" :endDate="endDate" trigger-thread/>
    </card-view>
</template>

<style lang="scss" scoped>

.card.keyword > .container > * {
    width: 100%;
    box-sizing: border-box;

    &.metadata {
        font-size: 2.4vh;
        padding: 0.6vh 0.6vw;
        white-space: nowrap;
        overflow: hidden;
        text-overflow: ellipsis;
    }

    &:not(:last-child) {
        border-bottom: 1px solid #ebeef5;
    }

    &.line-chart { height: 20vh }
    &.word-cloud { height: 24vh }
    &.bar-chart { height: 42vh }
}

</style>
