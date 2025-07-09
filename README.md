# import re
from collections import defaultdict
from textblob import TextBlob

class SentimentSummarizer:
    def __init__(self, summary_ratio=0.3):
        self.summary_ratio = summary_ratio
        self.sentiment_scores = defaultdict(float)

    def _split_sentences(self, text):
        # Improved sentence splitting with regex
        sentences = re.split(r'(?<!\w\.\w.)(?<![A-Z][a-z]\.)(?<=\.|\?|\!)\s', text)
        return [s.strip() for s in sentences if s.strip()]

    def _calculate_sentiment_weights(self, sentences):
        for idx, sentence in enumerate(sentences):
            blob = TextBlob(sentence)
            # Prioritize extreme sentiments (both positive and negative)
            self.sentiment_scores[idx] = abs(blob.sentiment.polarity)

    def _rank_sentences(self, sentences):
        # Combine sentiment score with sentence position (earlier sentences weighted slightly)
        ranked = []
        for idx, sentence in enumerate(sentences):
            position_weight = 1.0 - (idx / len(sentences)) * 0.2  # 20% position weight
            total_score = self.sentiment_scores[idx] * 0.8 + position_weight
            ranked.append((-total_score, idx))  # Negative for ascending sort
        
        ranked.sort()
        return [idx for (score, idx) in ranked]

    def summarize(self, text):
        sentences = self._split_sentences(text)
        if len(sentences) <= 3:
            return ' '.join(sentences)
            
        self._calculate_sentiment_weights(sentences)
        ranked_indices = self._rank_sentences(sentences)
        
        summary_length = max(2, int(len(sentences) * self.summary_ratio))
        top_indices = sorted(ranked_indices[:summary_length])
        
        return ' '.join([sentences[i] for i in top_indices])

# Example usage
if __name__ == "__main__":
    sample_text = """
    The new restaurant downtown has amazing ambiance! The decor is stunning and the staff are incredibly friendly.
    However, the food was quite disappointing. My steak was overcooked and the pasta was underwhelming.
    Despite the food issues, I might return just for the excellent service and cozy atmosphere.
    The dessert, a chocolate lava cake, was absolutely delicious and made up for some of the main course flaws.
    """
    
    summarizer = SentimentSummarizer(summary_ratio=0.4)
    print("Original Text:\n", sample_text)
    print("\nSummary:\n", summarizer.summarize(sample_text))
